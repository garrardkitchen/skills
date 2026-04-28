# EF Core Concurrency, Transactions, and Caching

Use this file when the user asks about consistency, conflicts, or distributed caching.

## Concurrency token

```csharp
public sealed class Order
{
    public Guid Id { get; set; }
    public string Status { get; set; } = string.Empty;

    [Timestamp]
    public byte[] RowVersion { get; set; } = [];
}
```

Concurrency tokens SHOULD be used for records where lost updates would affect money, permissions, personal data, workflow state, approvals, or externally visible actions.

## Handling concurrency conflicts

```csharp
try
{
    await db.SaveChangesAsync(cancellationToken);
}
catch (DbUpdateConcurrencyException)
{
    return Results.Conflict(new ProblemDetails
    {
        Title = "The resource was modified by another operation.",
        Status = StatusCodes.Status409Conflict
    });
}
```

For HTTP APIs, consider `ETag` / `If-Match` with `412 Precondition Failed` when clients update a known representation version.

## Transaction boundary

```csharp
await using var transaction = await db.Database.BeginTransactionAsync(cancellationToken);

order.Status = "Completed";
await db.SaveChangesAsync(cancellationToken);

db.OutboxMessages.Add(new OutboxMessage { Id = Guid.NewGuid(), Type = "OrderCompleted", Payload = "{}" });
await db.SaveChangesAsync(cancellationToken);

await transaction.CommitAsync(cancellationToken);
```

## Distributed cache with invalidation

```csharp
using Microsoft.Extensions.Caching.Distributed;

public sealed class CachedOrderReader(IDistributedCache cache, AppDbContext db)
{
    public async Task<string?> GetOrderStatusAsync(Guid orderId, CancellationToken cancellationToken)
    {
        string key = $"orders:{orderId}";
        string? cached = await cache.GetStringAsync(key, cancellationToken);
        if (cached is not null) return cached;

        string? status = await db.Orders
            .AsNoTracking()
            .Where(x => x.Id == orderId)
            .Select(x => x.Status)
            .SingleOrDefaultAsync(cancellationToken);

        if (status is not null)
        {
            await cache.SetStringAsync(
                key,
                status,
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
                },
                cancellationToken);
        }

        return status;
    }
}
```

## Cache safety rules

- every cache entry SHOULD have a TTL
- include tenant/user scope in cache keys when data is tenant- or user-specific
- invalidate or refresh cache entries after writes
- protect against cache stampedes for expensive reads
- do not cache secrets, bearer tokens, or unredacted personal data unless the cache is explicitly approved for that data class
- treat cached authorization decisions as short-lived and revocable

## Sources

- Microsoft Learn EF Core concurrency guidance
- Microsoft Learn EF Core transactions and performance guidance
- OWASP API Security Top 10 2023 API3 and API4
