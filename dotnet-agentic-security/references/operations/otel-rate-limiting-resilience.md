# OpenTelemetry, Rate Limiting, and Resilience

Use this file for operational guidance.

## OpenTelemetry

```csharp
using OpenTelemetry.Resources;

builder.Services.AddOpenTelemetry()
    .ConfigureResource(resource => resource.AddService("MyApp.Api"))
    .WithTracing(tracing =>
    {
        tracing
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddEntityFrameworkCoreInstrumentation();
    })
    .WithMetrics(metrics =>
    {
        metrics
            .AddAspNetCoreInstrumentation()
            .AddRuntimeInstrumentation()
            .AddHttpClientInstrumentation();
    });
```

## Rate limiting

```csharp
using System.Globalization;
using System.Threading.RateLimiting;
using Microsoft.AspNetCore.RateLimiting;

builder.Services.AddRateLimiter(options =>
{
    options.RejectionStatusCode = StatusCodes.Status429TooManyRequests;
    options.OnRejected = (context, cancellationToken) =>
    {
        if (context.Lease.TryGetMetadata(MetadataName.RetryAfter, out TimeSpan retryAfter))
        {
            context.HttpContext.Response.Headers.RetryAfter =
                ((int)retryAfter.TotalSeconds).ToString(CultureInfo.InvariantCulture);
        }

        return ValueTask.CompletedTask;
    };

    options.AddPolicy("api", context =>
    {
        string partitionKey =
            context.User.Identity?.Name
            ?? context.Connection.RemoteIpAddress?.ToString()
            ?? "anonymous";

        return RateLimitPartition.GetFixedWindowLimiter(partitionKey, _ => new FixedWindowRateLimiterOptions
        {
            PermitLimit = 100,
            Window = TimeSpan.FromMinutes(1),
            QueueLimit = 0
        });
    });
});

app.UseRateLimiter();

RouteGroupBuilder api = app.MapGroup("/api")
    .RequireRateLimiting("api")
    .RequireAuthorization();

api.MapGet("/orders/{id:guid}", GetOrderAsync);
```

## Resilience

- prefer bounded retries
- use timeouts
- add circuit breakers for fragile downstreams
- avoid retry storms on non-idempotent writes
- emit retry, timeout, circuit-breaker, and rate-limit metrics
- return `429` with `Retry-After` where clients can act on it
- partition limits by user, tenant, client app, tool, or IP depending on the abuse model

## Guidance

- correlate logs/traces/metrics
- propagate trace context across queues or workflow boundaries
- capture downstream status codes and latency
- redact secrets, tokens, personal data, prompts, retrieved content, and tool payloads before exporting telemetry
- include service name, environment, version, tenant where appropriate, and correlation IDs
- sample traces deliberately; do not sample away all security-relevant failures

## Sources

- Microsoft Learn OpenTelemetry with .NET
- Microsoft Learn ASP.NET Core rate limiting
- Microsoft Learn .NET HTTP resilience
- OpenTelemetry semantic conventions
- OWASP API Security Top 10 2023 API4 and API6
