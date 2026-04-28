# Authorization and Access Control in ASP.NET Core

Use this file when the user asks about authorization, access control, BOLA, BFLA, ownership checks, privileged actions, app roles, scopes, or tenant-aware authorization.

Access control is the first implementation priority for most APIs. Authentication proves who or what called the app; authorization proves the caller can perform this action on this resource in this tenant.

## Must implement

- Require authentication for non-public endpoints.
- Use policy-based authorization for privileged routes and operations.
- Enforce resource-level authorization for every endpoint that accepts caller-controlled object IDs.
- Validate tenant membership against the application's tenant model, not just the token issuer.
- Separate Azure RBAC, Entra directory roles, Microsoft Graph permissions, application roles, delegated scopes, and business permissions.
- Deny by default when ownership, tenant, or policy evidence is missing.

## Resource-based authorization

```csharp
public sealed class CanReadOrderRequirement : IAuthorizationRequirement
{
}

public sealed class CanReadOrderHandler(ICurrentTenant currentTenant)
    : AuthorizationHandler<CanReadOrderRequirement, Order>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        CanReadOrderRequirement requirement,
        Order resource)
    {
        string? objectId = context.User.FindFirst("oid")?.Value;
        string? tenantId = currentTenant.TenantId;

        if (resource.TenantId == tenantId &&
            (resource.OwnerObjectId == objectId || HasAppRole(context.User, "Orders.Admin")))
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }

    private static bool HasAppRole(ClaimsPrincipal user, string role) =>
        // Requires JWT bearer configuration with MapInboundClaims = false or RoleClaimType = "roles".
        user.FindAll("roles").Any(claim => claim.Value == role);
}
```

Resolve the application's tenant from your membership model or tenant-routing layer. Do not treat the Entra `tid` claim as the application tenant unless your product explicitly models issuer tenant and application tenant as the same boundary.

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("Orders.ResourceRead", policy =>
        policy.Requirements.Add(new CanReadOrderRequirement()));
});

builder.Services.AddScoped<IAuthorizationHandler, CanReadOrderHandler>();
```

## Endpoint pattern

```csharp
app.MapGet("/api/orders/{id:guid}", async (
    Guid id,
    ClaimsPrincipal user,
    IAuthorizationService authorization,
    IOrderRepository orders,
    CancellationToken cancellationToken) =>
{
    Order? order = await orders.GetAsync(id, cancellationToken);
    if (order is null)
        return Results.NotFound();

    AuthorizationResult allowed = await authorization.AuthorizeAsync(user, order, "Orders.ResourceRead");
    return allowed.Succeeded ? Results.Ok(OrderDto.From(order)) : Results.Forbid();
})
.RequireAuthorization();
```

## Verification

- Missing token returns `401`.
- Valid token without required route policy returns `403`.
- Valid token for user A cannot access user B's object.
- Valid token for tenant A cannot access tenant B's object.
- Admin role has only documented admin capabilities.
- App-only token cannot call user-delegated endpoints unless explicitly allowed.

## Avoid

- Checking only `User.Identity.IsAuthenticated`.
- Treating `Owner` or `Contributor` Azure RBAC as application authorization.
- Trusting route IDs without loading and authorizing the resource.
- Returning `404` for every authorization failure without a deliberate information-disclosure decision.
- Hiding authorization checks in generic repositories where reviewers cannot see them.

## Sources

- Microsoft Learn ASP.NET Core policy and resource-based authorization
- OWASP Authorization Cheat Sheet
- OWASP API Security Top 10 2023 API1 and API5
- OWASP ASVS v5.0.0 access control requirements
