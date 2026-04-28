# OWASP Top Ten Web Application Security Risks and API Mitigations for ASP.NET Core

Use this file when the user asks for web application security guidance, API security, or classic OWASP mitigations.

This file maps OWASP Top Ten Web Application Security Risks 2025 (`A01:2025` through `A10:2025`) and OWASP API Security Top 10 2023 risk families to practical ASP.NET Core mitigations. Treat access control, API object authorization, and tenant isolation as first-priority implementation concerns. Include the OWASP code beside example code when a snippet addresses a specific risk.

## Web risk-to-mitigation map

| Code | Risk family | ASP.NET Core mitigation focus |
|---|---|---|
| `A01:2025` | Broken Access Control | Policy-based authorization, ownership checks, route-level and resource-level enforcement |
| `A02:2025` | Security Misconfiguration | Environment separation, secure headers, production-safe middleware, locked-down diagnostics |
| `A03:2025` | Software Supply Chain Failures | Package hygiene, pinned versions, source mapping, SBOM/dependency review |
| `A04:2025` | Cryptographic Failures | HTTPS everywhere, Data Protection, Key Vault, modern algorithms, no plaintext secrets |
| `A05:2025` | Injection | Parameterized queries, safe LINQ, validated input, output encoding, no dynamic SQL from raw input |
| `A06:2025` | Insecure Design | Threat modeling, trust boundaries, secure-by-default endpoints, least privilege |
| `A07:2025` | Authentication Failures | OIDC/JWT, MFA, session/token hygiene, no custom auth protocols |
| `A08:2025` | Software or Data Integrity Failures | Trusted package sources, signed artifacts, protected CI/CD, config integrity |
| `A09:2025` | Security Logging and Alerting Failures | Structured logs, audit trails, correlation IDs, alert-worthy failures |
| `A10:2025` | Mishandling of Exceptional Conditions | Problem Details, production exception handling, no stack traces to callers |

## API Security Top 10 2023 map

| API risk | ASP.NET Core implementation priority |
|---|---|
| `API1:2023` Broken Object Level Authorization | Check resource ownership or tenant membership in every handler using caller-provided IDs |
| `API2:2023` Broken Authentication | Validate issuer, audience, lifetime, signing keys, token type, and expected flow |
| `API3:2023` Broken Object Property Level Authorization | Use narrow DTOs; ignore or reject over-posted properties; enforce field-level authorization where needed |
| `API4:2023` Unrestricted Resource Consumption | Add request limits, pagination, rate limits, quotas, cancellation, and bounded downstream calls |
| `API5:2023` Broken Function Level Authorization | Require policies for privileged endpoints and administrative flows |
| `API6:2023` Unrestricted Access to Sensitive Business Flows | Add abuse controls, bot/rate protections, workflow quotas, and business-level monitoring |
| `API7:2023` SSRF | Use outbound allowlists, block internal metadata/private ranges, and avoid arbitrary user-controlled fetches |
| `API8:2023` Security Misconfiguration | Lock down Swagger, diagnostics, CORS, headers, environment settings, and management endpoints |
| `API9:2023` Improper Inventory Management | Maintain endpoint/version inventory and retire deprecated APIs deliberately |
| `API10:2023` Unsafe Consumption of APIs | Validate third-party API responses and authenticate/authorize callbacks/webhooks |

## Practical mitigation snippets

### `A01:2025` Broken Access Control

```csharp
// Delegated user-token example. App-only callers should use a separate `roles` policy.
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("Orders.ScopeRead", policy =>
        policy.RequireAssertion(context => HasScope(context.User, "orders.read")));
});

static bool HasScope(ClaimsPrincipal user, string scope) =>
    user.FindFirst("scp")?.Value
        .Split(' ', StringSplitOptions.RemoveEmptyEntries)
        .Contains(scope, StringComparer.Ordinal) == true;

app.MapGet("/orders/{id:guid}", async (Guid id, ClaimsPrincipal user, IOrderReader reader) =>
{
    Order? order = await reader.GetAsync(id);
    if (order is null) return Results.NotFound();

    if (order.OwnerObjectId != user.FindFirst("oid")?.Value)
        return Results.Forbid();

    return Results.Ok(OrderDto.From(order));
})
.RequireAuthorization("Orders.ScopeRead");
```

When using Microsoft Entra ID access tokens with `MapInboundClaims = false`, prefer the `scp` claim name for delegated scopes.

For app-only tokens, validate `roles` or app permissions with a separate policy instead of `scp`. For every endpoint that accepts an object ID, enforce resource ownership or tenant membership in application code; endpoint-level authentication is not enough.

### `A04:2025` Cryptographic Failures

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

### `A05:2025` Injection

```csharp
// Good: parameterized by EF Core
IReadOnlyList<Customer> customers = await db.Customers
    .Where(c => c.Email == request.Email)
    .ToListAsync(cancellationToken);

// Avoid building SQL from untrusted strings.
```

### `A06:2025` Insecure Design

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

### `A02:2025` Security Misconfiguration

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

### `A03:2025` Software Supply Chain Failures

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

### `A07:2025` Authentication Failures

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

### `A08:2025` Software or Data Integrity Failures

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

### `A09:2025` Security Logging and Alerting Failures

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

### `API7:2023` SSRF

```csharp
// Validation gate only. Pair with egress firewall/proxy controls or a
// rebinding-resistant HTTP handler before sending the outbound request.
using System.Net;
using System.Net.Sockets;

public static class SafeUriPolicy
{
    private static readonly HashSet<string> AllowedHosts = new(StringComparer.OrdinalIgnoreCase)
    {
        "api.contoso.com",
        "graph.microsoft.com"
    };

    public static async Task<Uri> ValidateExternalUriAsync(string candidate, CancellationToken cancellationToken)
    {
        if (!Uri.TryCreate(candidate, UriKind.Absolute, out Uri? uri))
            throw new InvalidOperationException("Invalid URI.");

        if (uri.Scheme != Uri.UriSchemeHttps || (!uri.IsDefaultPort && uri.Port != 443))
            throw new InvalidOperationException("Only HTTPS destinations on port 443 are allowed.");

        if (!AllowedHosts.Contains(uri.Host))
            throw new InvalidOperationException("Host not allowlisted.");

        IPAddress[] addresses = await Dns.GetHostAddressesAsync(uri.IdnHost, cancellationToken);
        if (addresses.Length == 0 || addresses.Any(IsPrivateOrLoopback))
            throw new InvalidOperationException("Destination resolves to a private or loopback address.");

        return uri;
    }

    private static bool IsPrivateOrLoopback(IPAddress address)
    {
        if (IPAddress.IsLoopback(address))
            return true;

        if (address.AddressFamily == AddressFamily.InterNetwork)
        {
            byte[] bytes = address.GetAddressBytes();
            return bytes[0] == 10
                || bytes[0] == 127
                || (bytes[0] == 172 && bytes[1] >= 16 && bytes[1] <= 31)
                || (bytes[0] == 192 && bytes[1] == 168)
                || (bytes[0] == 169 && bytes[1] == 254);
        }

        if (address.AddressFamily == AddressFamily.InterNetworkV6)
        {
            byte[] bytes = address.GetAddressBytes();
            bool isUniqueLocal = (bytes[0] & 0xfe) == 0xfc;
            return address.IsIPv6LinkLocal || address.IsIPv6SiteLocal || isUniqueLocal;
        }

        return false;
    }
}

builder.Services.AddHttpClient("approved-outbound")
    .ConfigurePrimaryHttpMessageHandler(() => new HttpClientHandler
    {
        AllowAutoRedirect = false
    });
```

An outbound allowlist MUST include scheme, host, port, and destination class checks where SSRF matters. DNS validation alone is not a complete SSRF defense because the actual connection can resolve the hostname again; pair application checks with rebinding-resistant egress controls such as an approved outbound proxy, firewall rules, or a handler that pins the validated destination. Disable automatic redirects; if redirects are intentionally allowed, validate each `Location` target with the same policy before following it. Do not allow user input to reach internal metadata endpoints, private address ranges, or arbitrary redirects.

## Coverage check

This file includes practical mitigation guidance for OWASP Top Ten Web Application Security Risks 2025 and OWASP API Security Top 10 2023 risk families:

1. `A01:2025` Broken Access Control
2. `A02:2025` Security Misconfiguration
3. `A03:2025` Software Supply Chain Failures
4. `A04:2025` Cryptographic Failures
5. `A05:2025` Injection
6. `A06:2025` Insecure Design
7. `A07:2025` Authentication Failures
8. `A08:2025` Software or Data Integrity Failures
9. `A09:2025` Security Logging and Alerting Failures
10. `A10:2025` Mishandling of Exceptional Conditions
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

- OWASP Top Ten Web Application Security Risks 2025 (`A01:2025` through `A10:2025`)
- OWASP API Security Top 10 2023
- OWASP ASVS v5.0.0
- OWASP Authorization, Authentication, REST, and .NET Security Cheat Sheets
- Microsoft Learn ASP.NET Core security documentation
