# Service Bus, Webhooks, and Distributed Correlation

Use this file when the user asks about queues, asynchronous processing, or replay protection.

## Service Bus processor

```csharp
using Azure.Core;
using Azure.Identity;
using Azure.Messaging.ServiceBus;

ILogger logger = loggerFactory.CreateLogger("OrdersProcessor");

string? managedIdentityClientId = configuration["Azure:ManagedIdentityClientId"];
TokenCredential credential = string.IsNullOrWhiteSpace(managedIdentityClientId)
    ? new ManagedIdentityCredential()
    : new ManagedIdentityCredential(managedIdentityClientId);

await using ServiceBusClient client = new("namespace.servicebus.windows.net", credential);
await using ServiceBusProcessor processor = client.CreateProcessor("orders", new ServiceBusProcessorOptions
{
    AutoCompleteMessages = false,
    MaxConcurrentCalls = 4
});

processor.ProcessMessageAsync += async args =>
{
    string body = args.Message.Body.ToString();
    // Validate, authorize, process idempotently, and persist side effects before completing.
    await args.CompleteMessageAsync(args.Message);
};

processor.ProcessErrorAsync += args =>
{
    logger.LogError(args.Exception, "Service Bus processing failed for {EntityPath}", args.EntityPath);
    return Task.CompletedTask;
};
```

Do not complete messages before validation and durable side effects succeed. For long-running work, handle lock renewal and cancellation deliberately.

## Dead-letter handling

```csharp
await receiver.DeadLetterMessageAsync(message, "validation-failed", "Payload did not pass validation.");
```

Use dead-lettering for poison messages after bounded retry attempts. Include safe reason codes, not secrets or full payloads.

## Webhook replay protection

```csharp
public sealed class ReplayGuard(IDistributedCache cache)
{
    public async Task<bool> IsReplayAsync(string messageId, CancellationToken cancellationToken)
    {
        string key = $"webhook-replay:{messageId}";
        string? existing = await cache.GetStringAsync(key, cancellationToken);
        if (existing is not null) return true;

        await cache.SetStringAsync(
            key,
            "1",
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            },
            cancellationToken);
        return false;
    }
}
```

For high-risk webhooks, prefer an atomic insert into a durable store with a unique message ID where the cache API cannot guarantee atomic add semantics.

## Distributed correlation

```csharp
using System.Diagnostics;

ActivitySource source = new("MyApp.Workflows");

using Activity? activity = source.StartActivity("ProcessOrderMessage");
activity?.SetTag("messaging.system", "azure_service_bus");
activity?.SetTag("messaging.destination", "orders");
```

## Messaging safety rules

- validate message schema before processing
- make handlers idempotent
- propagate correlation and trace context
- include tenant and subject context where applicable
- use bounded concurrency and backpressure
- use dead-letter handling for poison messages
- do not put secrets or unnecessary personal data in messages
- verify webhook signatures, timestamp tolerance, replay protection, and body limits

## Sources

- Microsoft Learn Azure Service Bus processor and settlement guidance
- OWASP REST Security Cheat Sheet
- OWASP API Security Top 10 2023 API4 and API10
- OpenTelemetry semantic conventions for messaging
