# Identity, MFA, RBAC, and App Roles

Use this file when the user asks about authentication or authorization.

## Core rules

- use the Microsoft identity platform or another established OIDC provider
- enforce MFA for human operators through the identity provider
- separate Azure RBAC from application authorization
- use app roles, scopes, or policies inside the application
- validate issuer, audience, lifetime, signing keys, and expected token type
- distinguish delegated user scopes (`scp`) from app-only permissions (`roles`)
- enforce resource ownership and tenant membership after endpoint-level authorization

## JWT bearer setup

```csharp
builder.Services
    .AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.Authority = builder.Configuration["Auth:Authority"];
        options.Audience = builder.Configuration["Auth:Audience"];
        options.MapInboundClaims = false;
    });

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("Devices.Read", policy =>
        policy.RequireAssertion(context => HasScope(context.User, "devices.read")));
    options.AddPolicy("Devices.AppRead", policy =>
        policy.RequireClaim("roles", "Devices.Read.All"));
});

static bool HasScope(ClaimsPrincipal user, string scope) =>
    user.FindFirst("scp")?.Value
        .Split(' ', StringSplitOptions.RemoveEmptyEntries)
        .Contains(scope, StringComparer.Ordinal) == true;
```

## Policy-based authorization

```csharp
app.MapPost("/api/v1/devices/{id:guid}/retire", async (...) =>
{
    // handler omitted
})
.RequireAuthorization("Devices.Write");
```

## Guidance

- use claims/policies close to business intent
- do not treat Azure Contributor or Owner as application roles
- check resource ownership where relevant
- keep operator and workload identities separate
- do not rely on route-level `[Authorize]` or `.RequireAuthorization()` alone for object-level decisions
- for multi-tenant apps, validate the tenant claim against the resource tenant and the app's tenancy model

## MFA

- MFA is enforced by the identity provider, not by ad hoc API code
- app code should still enforce authorization and protect high-impact actions
- for sensitive approval operations, require stronger policy or re-auth at the front end or identity layer

## Minimum security tests

- missing token returns `401`
- valid token without required policy returns `403`
- user with broad access cannot access another user's resource
- tenant A token cannot access tenant B data
- app-only token cannot call user-delegated endpoints unless explicitly allowed
- delegated token cannot call app-only administrative endpoints

## Sources

- Microsoft Learn ASP.NET Core policy-based authorization
- Microsoft identity platform access token guidance
- OWASP Authorization Cheat Sheet
- OWASP API Security Top 10 2023 API1 and API5
