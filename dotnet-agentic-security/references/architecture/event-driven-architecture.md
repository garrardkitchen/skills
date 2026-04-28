# Event-Driven Architecture in .NET

Use this file when the user asks about events, workflows, outbox patterns, or integration boundaries.

## When EDA helps

- background workflows
- cross-service integration
- eventual consistency
- audit and notification pipelines
- fan-out processing with bounded consumers

## Rules

- distinguish domain events from integration events
- make handlers idempotent
- persist state changes and event publication safely
- add correlation IDs and trace context
- validate event schema and version before processing
- do not put secrets, access tokens, or unnecessary personal data in event payloads
- use dead-letter handling and bounded retries instead of infinite loops

## Outbox pattern

```csharp
public sealed class OutboxMessage
{
    public Guid Id { get; init; }
    public string Type { get; init; } = default!;
    public string Payload { get; init; } = default!;
    public DateTimeOffset OccurredUtc { get; init; }
}
```

```csharp
public async Task SaveOrderAndOutboxAsync(Order order, OutboxMessage message, CancellationToken cancellationToken)
{
    db.Orders.Add(order);
    db.OutboxMessages.Add(message);
    await db.SaveChangesAsync(cancellationToken);
}
```

## Idempotent handler

```csharp
public sealed class OrderCreatedHandler(AppDbContext db, ILogger<OrderCreatedHandler> logger)
{
    public async Task HandleAsync(OrderCreatedIntegrationEvent message, CancellationToken cancellationToken)
    {
        bool alreadyProcessed = await db.ProcessedMessages
            .AnyAsync(x => x.MessageId == message.MessageId, cancellationToken);

        if (alreadyProcessed)
            return;

        await using var transaction = await db.Database.BeginTransactionAsync(cancellationToken);

        db.ProcessedMessages.Add(new ProcessedMessage
        {
            MessageId = message.MessageId,
            ProcessedUtc = DateTimeOffset.UtcNow
        });

        await db.SaveChangesAsync(cancellationToken);
        await transaction.CommitAsync(cancellationToken);

        logger.LogInformation("Processed message {MessageId}", message.MessageId);
    }
}
```

Use a database unique constraint on `MessageId` for stronger duplicate protection under concurrency.

## Best practices

- keep event contracts versionable
- do not put secrets in event payloads
- use retries with dead-letter handling, not infinite loops
- cap fan-out and protect downstream dependencies
- for externally visible workflows, add compensating actions or reconciliation
- include tenant and subject context where required for authorization and audit
- propagate W3C trace context or equivalent correlation across service boundaries
- version contracts; do not break consumers silently

## Verification

- duplicate message is ignored or handled idempotently
- poison message is dead-lettered after bounded attempts
- event payload does not include secrets or unnecessary personal data
- trace/correlation ID appears in logs across publisher and consumer
- tenant A event cannot mutate tenant B state

## Sources

- Microsoft Learn Azure Service Bus messaging guidance
- Microsoft Learn EF Core transactions
- OpenTelemetry semantic conventions for messaging
- OWASP API Security Top 10 2023 API4 and API10
