# HTTP API Best Practices for ASP.NET Core

Use this file when the user is building or reviewing HTTP APIs.

## API design defaults

- use explicit versioning
- validate request models at the boundary
- return `ProblemDetails` for errors
- document with OpenAPI
- paginate list endpoints
- support idempotency where clients may retry writes
- use authn/authz by default for non-public endpoints
- enforce resource-level authorization for every endpoint that accepts caller-controlled object IDs
- use narrow request/response DTOs to avoid over-posting and object property authorization failures
- apply rate limits and quotas to expensive or abuse-prone business flows

## Versioning

Prefer URL or header versioning, but be consistent.

```csharp
app.MapGroup("/api/v1/orders")
   .RequireAuthorization()
   .MapGet("/{id:guid}", async (Guid id, IOrderReader reader, CancellationToken ct) =>
   {
       OrderDto? order = await reader.GetAsync(id, ct);
       return order is null ? Results.NotFound() : Results.Ok(order);
   });
```

## Validation

```csharp
public sealed record CreateOrderRequest(Guid CustomerId, IReadOnlyList<CreateOrderLineRequest> Lines);

public sealed class CreateOrderValidator : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderValidator()
    {
        RuleFor(x => x.CustomerId).NotEmpty();
        RuleFor(x => x.Lines).NotEmpty();
    }
}
```

Validation is not authorization. A syntactically valid ID or DTO can still reference another user's object or tenant.

## Problem Details

```csharp
builder.Services.AddProblemDetails();

app.UseExceptionHandler();
app.UseStatusCodePages();
```

## OpenAPI

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddOpenApi();

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}
```

Outside development, protect OpenAPI endpoints with authorization or publish static documentation through an approved internal path. Do not expose internal operation names, deprecated versions, or admin endpoints publicly by accident.

## Idempotency

- for externally retried writes, accept an idempotency key
- persist request outcome keyed by client + operation + key
- expire idempotency records after a defined retention window
- do not retry non-idempotent writes unless the operation is explicitly designed for replay safety

## Pagination

```csharp
app.MapGet("/api/v1/orders", async (
    int page,
    int pageSize,
    IOrderReader reader,
    CancellationToken ct) =>
{
    page = Math.Max(page, 1);
    pageSize = Math.Clamp(pageSize, 1, 100);

    return Results.Ok(await reader.ListAsync(page, pageSize, ct));
});
```

Prefer cursor or seek pagination for large or high-change datasets. Always cap `pageSize` or equivalent limits server-side.

## Resource authorization pattern

```csharp
app.MapGet("/api/v1/orders/{id:guid}", async (
    Guid id,
    ClaimsPrincipal user,
    IAuthorizationService authorization,
    IOrderReader reader,
    CancellationToken ct) =>
{
    OrderDto? order = await reader.GetAsync(id, ct);
    if (order is null)
        return Results.NotFound();

    AuthorizationResult allowed = await authorization.AuthorizeAsync(user, order, "CanReadOrder");
    return allowed.Succeeded ? Results.Ok(order) : Results.Forbid();
})
.RequireAuthorization();
```

Use resource-based authorization for BOLA-prone endpoints. A route-level policy proves the caller is authenticated or has a broad permission; it does not prove they can access the specific resource.

## Security notes

- never rely on obscurity for sensitive routes
- secure Swagger/OpenAPI endpoints in non-dev environments
- use outbound allowlists for callback or fetch features
- keep DTOs narrow and explicit
- test unauthorized, wrong-tenant, over-posted, replayed, and rate-limited requests

## Sources

- OWASP API Security Top 10 2023
- OWASP ASVS v5.0.0
- OWASP REST Security Cheat Sheet
- Microsoft Learn ASP.NET Core authorization, OpenAPI, Minimal API responses, and rate limiting
