# OWASP Web and API Mitigations for ASP.NET Core

Use this file when the user asks for web application security guidance, API security, or classic OWASP mitigations.

This file maps OWASP Top 10 2025 and OWASP API Security Top 10 2023 risk families to practical ASP.NET Core mitigations. Treat access control, API object authorization, and tenant isolation as first-priority implementation concerns.

## Web risk-to-mitigation map

| Risk family | ASP.NET Core mitigation focus |
|---|---|
| Broken access control | Policy-based authorization, ownership checks, route-level and resource-level enforcement |
| Security misconfiguration | Environment separation, secure headers, production-safe middleware, locked-down diagnostics |
| Software supply chain failures | Package hygiene, pinned versions, source mapping, SBOM/dependency review |
| Cryptographic failures | HTTPS everywhere, Data Protection, Key Vault, modern algorithms, no plaintext secrets |
| Injection | Parameterized queries, safe LINQ, validated input, output encoding, no dynamic SQL from raw input |
| Insecure design | Threat modeling, trust boundaries, secure-by-default endpoints, least privilege |
| Authentication failures | OIDC/JWT, MFA, session/token hygiene, no custom auth protocols |
| Software/data integrity failures | Trusted package sources, signed artifacts, protected CI/CD, config integrity |
| Security logging and alerting failures | Structured logs, audit trails, correlation IDs, alert-worthy failures |
| Mishandling exceptional conditions | Problem Details, production exception handling, no stack traces to callers |

## API Security Top 10 2023 map

| API risk | ASP.NET Core implementation priority |
|---|---|
| API1 Broken Object Level Authorization | Check resource ownership or tenant membership in every handler using caller-provided IDs |
| API2 Broken Authentication | Validate issuer, audience, lifetime, signing keys, token type, and expected flow |
| API3 Broken Object Property Level Authorization | Use narrow DTOs; ignore or reject over-posted properties; enforce field-level authorization where needed |
| API4 Unrestricted Resource Consumption | Add request limits, pagination, rate limits, quotas, cancellation, and bounded downstream calls |
| API5 Broken Function Level Authorization | Require policies for privileged endpoints and administrative flows |
| API6 Unrestricted Access to Sensitive Business Flows | Add abuse controls, bot/rate protections, workflow quotas, and business-level monitoring |
| API7 SSRF | Use outbound allowlists, block internal metadata/private ranges, and avoid arbitrary user-controlled fetches |
| API8 Security Misconfiguration | Lock down Swagger, diagnostics, CORS, headers, environment settings, and management endpoints |
| API9 Improper Inventory Management | Maintain endpoint/version inventory and retire deprecated APIs deliberately |
| API10 Unsafe Consumption of APIs | Validate third-party API responses and authenticate/authorize callbacks/webhooks |

## Practical mitigation snippets

### Broken access control

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("Orders.Read", policy => policy.RequireClaim("scp", "orders.read"));
});

app.MapGet("/orders/{id:guid}", async (Guid id, ClaimsPrincipal user, IOrderReader reader) =>
{
    OrderDto? order = await reader.GetAsync(id);
    if (order is null) return Results.NotFound();

    if (order.OwnerObjectId != user.FindFirst("oid")?.Value && !user.IsInRole("OrdersAdmin"))
        return Results.Forbid();

    return Results.Ok(order);
})
.RequireAuthorization("Orders.Read");
```

When using Microsoft Entra ID access tokens with `MapInboundClaims = false`, prefer the `scp` claim name for delegated scopes.

For app-only tokens, validate `roles` or app permissions instead of `scp`. For every endpoint that accepts an object ID, enforce resource ownership or tenant membership in application code; endpoint-level authentication is not enough.

### Cryptographic failures

```csharp
using Azure.Core;
using Azure.Identity;

string? managedIdentityClientId = builder.Configuration["Azure:ManagedIdentityClientId"];
TokenCredential credential = string.IsNullOrWhiteSpace(managedIdentityClientId)
    ? new ManagedIdentityCredential()
    : new ManagedIdentityCredential(managedIdentityClientId);

builder.Services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(new Uri(builder.Configuration["DataProtection:BlobUri"]!),
        credential)
    .ProtectKeysWithAzureKeyVault(new Uri(builder.Configuration["DataProtection:KeyIdentifier"]!),
        credential);
```

### Injection

```csharp
// Good: parameterized by EF Core
IReadOnlyList<Customer> customers = await db.Customers
    .Where(c => c.Email == request.Email)
    .ToListAsync(cancellationToken);

// Avoid building SQL from untrusted strings.
```

### Insecure design

```csharp
public sealed record TransferFundsRequest(Guid FromAccountId, Guid ToAccountId, decimal Amount);

app.MapPost("/transfers", async (
    TransferFundsRequest request,
    ClaimsPrincipal user,
    ITransferService transfers,
    CancellationToken cancellationToken) =>
{
    if (request.Amount <= 0)
        return Results.ValidationProblem(new Dictionary<string, string[]>
        {
            ["amount"] = ["Amount must be greater than zero."]
        });

    string? userObjectId = user.FindFirst("oid")?.Value;
    if (userObjectId is null)
        return Results.Forbid();

    bool ownsSourceAccount = await transfers.UserOwnsAccountAsync(userObjectId, request.FromAccountId, cancellationToken);
    if (!ownsSourceAccount)
        return Results.Forbid();

    await transfers.TransferAsync(request, cancellationToken);
    return Results.Accepted();
})
.RequireAuthorization();
```

### Security misconfiguration

```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

app.UseHttpsRedirection();

app.Use(async (context, next) =>
{
    context.Response.Headers["X-Content-Type-Options"] = "nosniff";
    context.Response.Headers["Referrer-Policy"] = "no-referrer";
    await next();
});
```

Never expose Swagger/OpenAPI, health details, developer exception pages, debug endpoints, or management endpoints publicly in production unless explicitly protected and approved.

### Vulnerable components

```xml
<!-- Directory.Packages.props -->
<Project>
  <ItemGroup>
    <PackageVersion Include="Azure.Identity" Version="1.13.2" />
    <PackageVersion Include="Microsoft.EntityFrameworkCore.SqlServer" Version="9.0.0" />
    <PackageVersion Include="Swashbuckle.AspNetCore" Version="7.2.0" />
  </ItemGroup>
</Project>
```

```bash
dotnet list package --vulnerable --include-transitive
```

For production systems, also use central package management, trusted package sources, dependency scanning, and SBOM generation where the delivery pipeline supports it.

### Authentication failures

```csharp
builder.Services
    .AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.Authority = builder.Configuration["Auth:Authority"];
        options.Audience = builder.Configuration["Auth:Audience"];
        options.MapInboundClaims = false;
    });
```

### Software and data integrity failures

```csharp
public sealed class WebhookSignatureValidator
{
    public static bool IsValid(string payload, string providedSignature, string secret)
    {
        if (string.IsNullOrWhiteSpace(providedSignature))
            return false;

        byte[] key = Encoding.UTF8.GetBytes(secret);
        byte[] data = Encoding.UTF8.GetBytes(payload);

        using var hmac = new HMACSHA256(key);
        string computed = Convert.ToHexString(hmac.ComputeHash(data));

        try
        {
            return CryptographicOperations.FixedTimeEquals(
                Convert.FromHexString(computed),
                Convert.FromHexString(providedSignature));
        }
        catch (FormatException)
        {
            return false;
        }
    }
}
```

```csharp
app.MapPost("/webhooks/provider", async (HttpRequest request, IConfiguration config) =>
{
    using var reader = new StreamReader(request.Body);
    string payload = await reader.ReadToEndAsync();
    string providedSignature = request.Headers["X-Signature"].ToString();

    if (!WebhookSignatureValidator.IsValid(payload, providedSignature, config["Webhook:Secret"]!))
        return Results.Unauthorized();

    return Results.Accepted();
});
```

Production webhook handlers SHOULD also verify provider-specific canonicalization, timestamp tolerance, replay protection, request body limits, and idempotency before acknowledging the event.

### Logging and monitoring failures

```csharp
app.Use(async (context, next) =>
{
    using var scope = app.Logger.BeginScope(new Dictionary<string, object?>
    {
        ["TraceId"] = context.TraceIdentifier,
        ["Path"] = context.Request.Path
    });

    await next();
});
```

### SSRF

```csharp
public static class SafeUriPolicy
{
    private static readonly HashSet<string> AllowedHosts =
    [
        "api.contoso.com",
        "graph.microsoft.com"
    ];

    public static Uri ValidateExternalUri(string candidate)
    {
        if (!Uri.TryCreate(candidate, UriKind.Absolute, out Uri? uri))
            throw new InvalidOperationException("Invalid URI.");

        if (!AllowedHosts.Contains(uri.Host))
            throw new InvalidOperationException("Host not allowlisted.");

        return uri;
    }
}
```

An outbound allowlist MUST include scheme, host, port, and destination class checks where SSRF matters. Do not allow user input to reach internal metadata endpoints, private address ranges, or arbitrary redirects.

## Coverage check

This file includes practical mitigation guidance for OWASP Top 10 2025 and OWASP API Security Top 10 2023 risk families:

1. Broken access control
2. Cryptographic failures
3. Injection
4. Insecure design
5. Injection
6. Insecure design
7. Authentication failures
8. Software and data integrity failures
9. Security logging and alerting failures
10. Mishandling exceptional conditions
11. API object/function/property authorization failures
12. API resource consumption, inventory, SSRF, and unsafe API consumption

## Guidance

- prefer built-in authn/authz primitives over custom schemes
- authenticate at the boundary, then authorize at route, function, resource, tenant, and property levels
- use `ProblemDetails` instead of leaking stack traces
- treat outbound HTTP as a security boundary
- keep diagnostics rich for operators but sparse for callers
- write negative tests for BOLA, BFLA, over-posting, SSRF, replay, and missing-auth cases

## Sources

- OWASP Top 10 2025
- OWASP API Security Top 10 2023
- OWASP ASVS v5.0.0
- OWASP Authorization, Authentication, REST, and .NET Security Cheat Sheets
- Microsoft Learn ASP.NET Core security documentation
