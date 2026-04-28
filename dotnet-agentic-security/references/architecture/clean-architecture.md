# Clean Architecture for .NET

Use this file when the user wants a maintainable solution structure.

## Recommended layers

- `Domain` — entities, value objects, domain services, domain events
- `Application` — use cases, commands, queries, validators, interfaces
- `Infrastructure` — EF Core, external APIs, queues, storage, Azure SDK clients
- `Web` or `Api` — controllers/endpoints, authn/authz, transport contracts

## Dependency rule

- outer layers depend inward
- domain should not depend on EF, ASP.NET Core, or Azure SDKs
- application can define interfaces implemented by infrastructure
- authorization decisions that depend on current user/resource context belong at use-case or endpoint boundary, not hidden in EF entities
- infrastructure must not bypass application-layer validation and authorization when executing commands

## Example solution layout

```text
src/
  MyApp.Api/
  MyApp.Application/
  MyApp.Domain/
  MyApp.Infrastructure/
tests/
  MyApp.UnitTests/
  MyApp.IntegrationTests/
```

## Endpoint to use-case boundary

```csharp
app.MapPost("/orders", async (
    CreateOrderRequest request,
    ClaimsPrincipal user,
    CreateOrderCommandHandler handler,
    CancellationToken cancellationToken) =>
{
    var result = await handler.HandleAsync(
        new CreateOrderCommand(request.CustomerId, request.Items),
        user,
        cancellationToken);

    return Results.Created($"/orders/{result.OrderId}", result);
})
.RequireAuthorization("Orders.Create");
```

Endpoints should authenticate and apply coarse policy. Use cases should still enforce business authorization, resource ownership, tenant membership, and invariants before mutation.

## Security boundary placement

| Concern | Preferred location | Why |
|---|---|---|
| JWT validation | Web/API | Transport boundary concern |
| Route policy | Web/API | Coarse endpoint access |
| Resource ownership | Application/use case | Needs loaded resource and business context |
| Entity invariants | Domain | Always true regardless of caller |
| EF provider details | Infrastructure | Framework-specific implementation |
| Secret retrieval | Infrastructure behind interface | Keeps SDK and credential details outside core logic |

## Common anti-patterns

- putting `HttpContext` into domain entities
- returning EF entities directly from API endpoints
- hiding authorization in generic repositories where reviewers cannot see it
- letting Infrastructure methods mutate state without an Application use case
- creating abstractions for every class before there is a real boundary to protect

## Best practices

- keep DTOs out of domain objects
- treat handlers/use cases as orchestration boundaries
- isolate infrastructure details behind interfaces only where abstraction is buying clarity
- do not create fake abstractions for everything
- prefer vertical slices when that is more understandable than deep layering
- make authorization, validation, and transaction boundaries visible in code
- write integration tests at the Web/API boundary and unit tests for use-case authorization outcomes

## Sources

- Microsoft Learn ASP.NET Core authorization and dependency injection guidance
- Microsoft Learn enterprise web app patterns
- OWASP ASVS v5.0.0 architecture and access control guidance
- OWASP API Security Top 10 2023 API1 and API5
