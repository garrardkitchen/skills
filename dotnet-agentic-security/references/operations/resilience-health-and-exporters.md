# Resilience Pipelines, Health Checks, and Exporters

Use this file when the user asks about production hardening.

## Standard resilience handler

```csharp
using Microsoft.Extensions.Http.Resilience;
using Polly;

builder.Services.AddHttpClient("DownstreamApi", client =>
{
    client.BaseAddress = new Uri("https://api.contoso.com/");
})
.AddStandardResilienceHandler()
.Configure(options =>
{
    options.TotalRequestTimeout.Timeout = TimeSpan.FromSeconds(30);
    options.Retry.MaxRetryAttempts = 3;
    options.Retry.Delay = TimeSpan.FromMilliseconds(500);
    options.Retry.BackoffType = DelayBackoffType.Exponential;
    options.CircuitBreaker.BreakDuration = TimeSpan.FromSeconds(10);
});
```

## OTLP exporter

```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddOtlpExporter();
    })
    .WithMetrics(metrics =>
    {
        metrics
            .AddAspNetCoreInstrumentation()
            .AddRuntimeInstrumentation()
            .AddOtlpExporter();
    });
```

## Health checks

```csharp
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy(), tags: ["live"])
    .AddDbContextCheck<AppDbContext>("sql", tags: ["ready"]);

app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("live")
});

app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
});
```

## Guidance

- use readiness for dependencies
- use liveness for process health
- export telemetry somewhere real, not just instrumentation only
- do not expose detailed dependency failure information publicly
- protect operational endpoints when they reveal topology or sensitive state
- align health checks with orchestrator behavior so failing dependencies do not cause restart loops unnecessarily
- configure resilience differently for idempotent reads and non-idempotent writes

## Sources

- Microsoft Learn .NET HTTP resilience
- Microsoft Learn ASP.NET Core health checks
- Microsoft Learn OpenTelemetry with .NET
- OWASP API Security Top 10 2023 API4
