# Token Validation and Claims

Use this file when the user asks about JWT bearer auth, Microsoft identity platform tokens, claims, scopes, roles, app-only tokens, delegated tokens, or multi-tenant token validation.

## Must implement

- Validate issuer, audience, signing keys, lifetime, and token type.
- Use HTTPS metadata endpoints.
- Disable inbound claim remapping unless the app intentionally depends on mapped claim names.
- Distinguish delegated scopes (`scp`) from app-only permissions (`roles`).
- Validate tenant and client application IDs for sensitive service-to-service paths.
- Treat token claims as authorization inputs, not as complete proof of resource access.

## JWT bearer setup

```csharp
builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = builder.Configuration["Auth:Authority"];
        options.Audience = builder.Configuration["Auth:Audience"];
        options.MapInboundClaims = false;
        options.TokenValidationParameters.ValidateIssuer = true;
        options.TokenValidationParameters.ValidateAudience = true;
        options.TokenValidationParameters.ValidateLifetime = true;
    });
```

## Delegated vs app-only policies

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("Orders.UserRead", policy =>
        policy.RequireAssertion(context => HasScope(context.User, "orders.read")));

    options.AddPolicy("Orders.AppRead", policy =>
        policy.RequireClaim("roles", "Orders.Read.All"));
});

static bool HasScope(ClaimsPrincipal user, string scope) =>
    user.FindFirst("scp")?.Value
        .Split(' ', StringSplitOptions.RemoveEmptyEntries)
        .Contains(scope, StringComparer.Ordinal) == true;
```

Use delegated scopes for user-on-behalf-of flows. Use app roles/application permissions for app-only service calls. Do not let a broad app-only token call user-specific endpoints unless the design explicitly allows it.

## Claims to handle deliberately

| Claim | Use |
|---|---|
| `iss` | trusted issuer validation |
| `aud` | API audience validation |
| `tid` | issuer tenant, not necessarily app tenant membership |
| `oid` | user or service principal object identifier |
| `azp` / `appid` | calling client application |
| `scp` | delegated user scopes |
| `roles` | app roles or application permissions |

## Verification

- wrong audience token is rejected
- wrong issuer token is rejected
- expired token is rejected
- delegated token without required `scp` is forbidden
- app-only token without required `roles` is forbidden
- token from unapproved client app is rejected for sensitive service APIs

## Avoid

- Accepting `alg: none` or relying on token header-selected algorithms.
- Using claims directly as database filters without resource authorization.
- Assuming `tid` equals the tenant in your SaaS domain model.
- Logging bearer tokens or full authorization headers.

## Sources

- Microsoft Learn ASP.NET Core JWT bearer authentication
- Microsoft identity platform access token claims
- OWASP REST Security and Authentication Cheat Sheets
