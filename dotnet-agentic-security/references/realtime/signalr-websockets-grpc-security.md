# SignalR, WebSockets, and gRPC Security

Use this file when the user asks about bidirectional or realtime transports.

## SignalR auth + CORS

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("RealtimeCors", policy =>
    {
        policy.WithOrigins("https://app.contoso.com")
              .WithMethods("GET", "POST")
              .AllowCredentials()
              .WithHeaders("Authorization", "Content-Type");
    });
});

builder.Services.AddSignalR();

app.MapHub<NotificationsHub>("/hubs/notifications")
   .RequireCors("RealtimeCors")
   .RequireAuthorization();
```

## gRPC-Web CORS exposure

```csharp
builder.Services.AddGrpc();

builder.Services.AddCors(options =>
{
    options.AddPolicy("GrpcWeb", policy =>
    {
        policy.WithOrigins("https://app.contoso.com")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .WithExposedHeaders("Grpc-Status", "Grpc-Message", "Grpc-Encoding", "Grpc-Accept-Encoding");
    });
});
```

## Guidance

- authenticate the connection, not just later method calls
- bound message size and frequency
- do not trust connection context without authorization checks
- authorize sensitive hub methods and gRPC methods individually
- avoid putting bearer tokens in query strings unless the transport requires it and logs are controlled
- disable detailed errors in production
- add per-user/tenant connection quotas
- validate message schemas and reject oversized payloads
- apply backpressure or disconnect slow consumers
- propagate trace/correlation IDs for long-lived streams

## Hub method authorization

```csharp
public sealed class NotificationsHub(ICurrentTenant currentTenant) : Hub
{
    [Authorize(Policy = "Notifications.Send")]
    public Task SendTenantNotification(string tenantId, string message)
    {
        string? callerTenant = currentTenant.TenantId;
        if (!string.Equals(callerTenant, tenantId, StringComparison.Ordinal))
            throw new HubException("Forbidden.");

        return Clients.Group(tenantId).SendAsync("notification", message);
    }
}
```

## Sources

- Microsoft Learn SignalR authentication and authorization
- Microsoft Learn gRPC security guidance
- OWASP API Security Top 10 2023 API1, API4, and API5
