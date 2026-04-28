# SSE and Streaming in ASP.NET Core

Use this file when the user asks for streaming APIs or Server-Sent Events.

## When SSE fits

- one-way server-to-client updates
- progress reporting
- agent or workflow streaming to a browser client

## Minimal SSE endpoint

```csharp
app.MapGet("/api/v1/stream", async (HttpContext context, CancellationToken cancellationToken) =>
{
    context.Response.Headers.ContentType = "text/event-stream";
    context.Response.Headers.CacheControl = "no-cache";

    for (int i = 0; i < 5 && !cancellationToken.IsCancellationRequested; i++)
    {
        await context.Response.WriteAsync($"event: progress\n", cancellationToken);
        await context.Response.WriteAsync($"data: {{\"step\": {i}}}\n\n", cancellationToken);
        await context.Response.Body.FlushAsync(cancellationToken);
        await Task.Delay(1000, cancellationToken);
    }
})
.RequireAuthorization();
```

## Security guidance

- require authorization for sensitive streams
- do not emit secrets or raw internal errors
- keep event payloads small and structured
- respect disconnects and cancellation
- throttle high-volume streams
- include tenant/user scope in queries that feed the stream
- avoid streaming raw model prompts, retrieved documents, tokens, or sensitive tool outputs
- set cache/proxy headers so intermediaries do not store sensitive events

## Operational guidance

- add heartbeat events for long-lived streams
- time out inactive sessions
- measure connected client count and stream duration
- understand proxy buffering and idle timeout behavior before production use
- enforce per-user or per-tenant stream quotas
- clean up server-side resources when clients disconnect

## Hardened headers

```csharp
context.Response.Headers.ContentType = "text/event-stream";
context.Response.Headers.CacheControl = "no-store";
context.Response.Headers.Pragma = "no-cache";
context.Response.Headers["X-Accel-Buffering"] = "no";
```

## Sources

- Microsoft Learn ASP.NET Core streaming and response guidance
- OWASP API Security Top 10 2023 API4
- OpenTelemetry .NET observability guidance
