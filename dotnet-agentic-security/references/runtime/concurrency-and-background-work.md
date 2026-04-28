# Concurrency, Locking, and Background Work

Use this file when the user asks about worker safety, locking, or concurrent coordination.

## `SemaphoreSlim` for bounded async access

```csharp
public sealed class DeviceRefreshCoordinator
{
    private readonly SemaphoreSlim _gate = new(initialCount: 4, maxCount: 4);

    public async Task RefreshAsync(Func<CancellationToken, Task> operation, CancellationToken cancellationToken)
    {
        await _gate.WaitAsync(cancellationToken);
        try
        {
            await operation(cancellationToken);
        }
        finally
        {
            _gate.Release();
        }
    }
}
```

## Guidance

- use `SemaphoreSlim` for async coordination
- avoid `lock` across async boundaries
- avoid unbounded `Task.WhenAll` over large collections
- persist progress for long-running jobs
- make background jobs idempotent where possible
- bound channels, queues, and in-memory work buffers
- propagate tenant, user, and correlation context into background work when it affects authorization or audit
- do not use fire-and-forget tasks for work that must be durable, observable, or retried

## Hosted service pattern

```csharp
public sealed class Worker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await DoWorkAsync(stoppingToken);
            await Task.Delay(TimeSpan.FromSeconds(10), stoppingToken);
        }
    }
}
```

## Good practices

- add cancellation everywhere
- bound retries
- emit metrics for queue depth, failures, and execution time
- separate scheduling from business work when complexity grows
- prefer `PeriodicTimer` over manual delay loops for periodic work where appropriate
- use distributed coordination or leases when multiple instances may process the same work
- record idempotency keys for externally triggered jobs

## Sources

- Microsoft Learn background tasks with hosted services
- Microsoft .NET secure coding guidelines for race conditions
- OWASP API Security Top 10 2023 API4
