# Multitenancy and Data Isolation

Use this file when the user asks about tenant safety, isolation, or row-level access control.

Tenant isolation is a security boundary. Endpoint authorization, EF query filters, cache keys, blob paths, queue messages, search indexes, logs, and background workers all need tenant-aware design.

## Tenant context propagation

```csharp
public interface ITenantContext
{
    string TenantId { get; }
}

public interface IAppTenantResolver
{
    string ResolveTenantId(ClaimsPrincipal user);
}

public sealed class TenantContext(IHttpContextAccessor accessor, IAppTenantResolver resolver) : ITenantContext
{
    public string TenantId =>
        resolver.ResolveTenantId(
            accessor.HttpContext?.User
            ?? throw new InvalidOperationException("Authenticated user is not available."));
}
```

The `tid` claim identifies the token issuer tenant in Microsoft Entra scenarios. It does not automatically prove the user belongs to an application tenant or can access a specific tenant resource. Validate the claim against your app's tenant membership model.

## EF Core query filter

```csharp
public sealed class AppDbContext(DbContextOptions<AppDbContext> options, ITenantContext tenantContext) : DbContext(options)
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>()
            .HasQueryFilter(x => x.TenantId == tenantContext.TenantId);
    }
}
```

Query filters are defense-in-depth, not the only tenant control. Write paths, raw SQL, background jobs, admin queries, cache lookups, and external storage access must also include tenant scope.

## Resource ownership checks

```csharp
app.MapGet("/api/orders/{id:guid}", async (
    Guid id,
    ClaimsPrincipal user,
    ITenantContext tenantContext,
    AppDbContext db,
    CancellationToken ct) =>
{
    string? objectId = user.FindFirst("oid")?.Value;

    Order? order = await db.Orders.SingleOrDefaultAsync(
        x => x.Id == id && x.TenantId == tenantContext.TenantId,
        ct);
    if (order is null) return Results.NotFound();
    if (order.OwnerObjectId != objectId && !user.IsInRole("TenantAdmin")) return Results.Forbid();

    return Results.Ok(order);
});
```

## Non-HTTP tenant propagation

- include tenant ID in queued messages and validate it before processing
- include tenant ID in cache keys, blob prefixes, search filters, and telemetry dimensions
- avoid using one broad admin identity to bypass tenant checks in background jobs
- keep per-tenant quotas and rate limits for expensive APIs, agents, and exports
- write negative tests proving tenant A cannot read, update, stream, export, or delete tenant B data

## Isolation options

| Model | Use when | Notes |
|---|---|---|
| Shared database, tenant column | Low/medium isolation with strong app controls | Requires strict predicates, tests, and cache/storage scoping |
| Shared database with database row-level security | Higher assurance for relational data | Still validate at app boundary and for non-database resources |
| Database/schema per tenant | Stronger blast-radius control | More operational complexity and migration orchestration |
| Subscription/resource per tenant | Regulated or high-isolation workloads | Highest operational cost; useful for strict data residency or customer isolation |

## Sources

- OWASP API Security Top 10 2023 API1 and API3
- OWASP ASVS v5.0.0 access control requirements
- Microsoft ASP.NET Core authorization guidance
- Microsoft EF Core query filters and multitenancy guidance
