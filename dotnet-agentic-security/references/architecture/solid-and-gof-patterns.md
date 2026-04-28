# SOLID and Security-Oriented Patterns in .NET

Use this file when the user asks for design patterns, maintainable object design, or where to put security-sensitive logic in a `.NET` application.

Patterns are useful only when they make risk boundaries clearer. Prefer simple code until a pattern helps isolate authorization, validation, persistence, orchestration, external SDKs, or operational policy.

## Is the pattern list enough?

No. A useful secure `.NET` pattern set needs more than classic GoF examples. For ASP.NET Core, Azure, and agentic systems, include:

- SOLID principles with concrete boundaries
- Strategy, Factory, Decorator, Adapter, Observer
- Command / command handler
- Pipeline behaviors
- Specification / policy objects
- Result pattern
- Options pattern
- Outbox pattern
- Authorization handler
- Repository only where it adds a useful seam

Avoid pattern theatre: do not add abstractions that hide important security checks or make code harder to test.

---

## SOLID in practice

### Single Responsibility Principle

Split transport, validation, authorization, orchestration, and persistence so each can be reviewed and tested independently.

```csharp
public sealed record RetireDeviceCommand(Guid DeviceId, string Reason);

public sealed class RetireDeviceHandler(
    IDeviceRepository devices,
    IAuthorizationService authorization)
{
    public async Task<Result> HandleAsync(
        RetireDeviceCommand command,
        ClaimsPrincipal user,
        CancellationToken cancellationToken)
    {
        Device? device = await devices.GetAsync(command.DeviceId, cancellationToken);
        if (device is null) return Result.NotFound();

        AuthorizationResult allowed = await authorization.AuthorizeAsync(user, device, "Devices.Retire");
        if (!allowed.Succeeded) return Result.Forbidden();

        device.Retire(command.Reason);
        await devices.SaveAsync(device, cancellationToken);
        return Result.Success();
    }
}
```

### Open/Closed Principle

Extend behavior through registration or composition instead of spreading `switch` statements across business logic.

```csharp
public interface IRiskScoringRule
{
    RiskScore Evaluate(Transaction transaction);
}

public sealed class HighValueTransferRule : IRiskScoringRule
{
    public RiskScore Evaluate(Transaction transaction) =>
        transaction.Amount > 10_000m ? RiskScore.High : RiskScore.Low;
}

public sealed class RiskScorer(IEnumerable<IRiskScoringRule> rules)
{
    public RiskScore Score(Transaction transaction) =>
        rules.Select(rule => rule.Evaluate(transaction)).Max();
}
```

### Liskov Substitution Principle

Avoid inheritance when derived types weaken a security contract. Prefer explicit capabilities.

```csharp
public interface IReadOnlyTool
{
    Task<ToolResult> ReadAsync(ToolRequest request, CancellationToken cancellationToken);
}

public interface IWriteTool : IReadOnlyTool
{
    Task<ToolResult> WriteAsync(ToolRequest request, CancellationToken cancellationToken);
}
```

A read-only tool should never inherit a base type that exposes write methods it cannot safely honor.

### Interface Segregation Principle

Expose focused interfaces so callers cannot accidentally receive more authority than they need.

```csharp
public interface IOrderReader
{
    Task<OrderDto?> GetAsync(Guid id, CancellationToken cancellationToken);
}

public interface IOrderWriter
{
    Task CancelAsync(Guid id, string reason, CancellationToken cancellationToken);
}
```

Read endpoints should depend on `IOrderReader`, not a broad `IOrderService` that can also mutate state.

### Dependency Inversion Principle

Keep framework and cloud SDK details out of core business rules.

```csharp
public interface ISecretReader
{
    Task<string> GetSecretAsync(string name, CancellationToken cancellationToken);
}

public sealed class KeyVaultSecretReader(SecretClient client) : ISecretReader
{
    public async Task<string> GetSecretAsync(string name, CancellationToken cancellationToken)
    {
        KeyVaultSecret secret = await client.GetSecretAsync(name, cancellationToken: cancellationToken);
        return secret.Value;
    }
}
```

---

## GoF patterns worth using carefully

### Strategy

Good for pluggable behavior such as provider-specific persistence, risk scoring, redaction, or approval policy.

```csharp
public interface IApprovalPolicy
{
    bool RequiresApproval(ToolInvocation invocation);
}

public sealed class DestructiveToolApprovalPolicy : IApprovalPolicy
{
    public bool RequiresApproval(ToolInvocation invocation) =>
        invocation.IsDestructive || invocation.TargetDataClassification == "Restricted";
}
```

### Factory

Useful when object creation depends on configuration or app shape. Keep factories small and deterministic.

```csharp
public interface IDbProviderFactory
{
    DbConnection Create(string connectionString);
}

public sealed class SqlConnectionFactory : IDbProviderFactory
{
    public DbConnection Create(string connectionString) => new SqlConnection(connectionString);
}
```

### Decorator

Good for audit, validation, authorization, caching, retry, and telemetry around handlers.

```csharp
public interface ICommandHandler<TCommand>
{
    Task<Result> HandleAsync(TCommand command, CancellationToken cancellationToken);
}

public sealed class AuditingCommandHandler<TCommand>(
    ICommandHandler<TCommand> inner,
    ILogger<AuditingCommandHandler<TCommand>> logger) : ICommandHandler<TCommand>
{
    public async Task<Result> HandleAsync(TCommand command, CancellationToken cancellationToken)
    {
        logger.LogInformation("Handling command {CommandType}", typeof(TCommand).Name);
        Result result = await inner.HandleAsync(command, cancellationToken);
        logger.LogInformation("Handled command {CommandType} with {Status}", typeof(TCommand).Name, result.Status);
        return result;
    }
}
```

Do not use decorators to hide authorization failures or swallow exceptions.

### Observer

Prefer domain events or mediator-style notifications over hand-written callback tangles.

```csharp
public sealed record DeviceRetired(Guid DeviceId, DateTimeOffset OccurredUtc);

public interface IDomainEventPublisher
{
    Task PublishAsync<TEvent>(TEvent domainEvent, CancellationToken cancellationToken);
}
```

Domain events should not contain secrets, tokens, or excessive personal data.

### Adapter

Use when wrapping external SDKs or legacy APIs behind app-specific contracts.

```csharp
public interface IBlobWriter
{
    Task WriteAsync(string name, BinaryData content, CancellationToken cancellationToken);
}

public sealed class AzureBlobWriter(BlobContainerClient container) : IBlobWriter
{
    public Task WriteAsync(string name, BinaryData content, CancellationToken cancellationToken) =>
        container.GetBlobClient(name).UploadAsync(content, overwrite: false, cancellationToken);
}
```

Adapters should normalize errors and enforce safe defaults, but they should not bypass authorization decisions owned by application use cases.

---

## Modern .NET application patterns

### Command / command handler

Use for state-changing operations with validation, authorization, idempotency, and audit.

```csharp
public sealed record CancelOrderCommand(Guid OrderId, string Reason, string IdempotencyKey);

public sealed class CancelOrderHandler(
    IOrderRepository orders,
    IAuthorizationService authorization)
{
    public async Task<Result> HandleAsync(
        CancelOrderCommand command,
        ClaimsPrincipal user,
        CancellationToken cancellationToken)
    {
        Order? order = await orders.GetAsync(command.OrderId, cancellationToken);
        if (order is null) return Result.NotFound();

        AuthorizationResult allowed = await authorization.AuthorizeAsync(user, order, "Orders.Cancel");
        if (!allowed.Succeeded) return Result.Forbidden();

        order.Cancel(command.Reason);
        await orders.SaveAsync(order, cancellationToken);
        return Result.Success();
    }
}
```

### Pipeline behavior

Use when validation, authorization, telemetry, and transaction boundaries must be consistently applied.

```csharp
public interface IPipelineBehavior<TRequest, TResponse>
{
    Task<TResponse> HandleAsync(
        TRequest request,
        Func<CancellationToken, Task<TResponse>> next,
        CancellationToken cancellationToken);
}

public sealed class ValidationBehavior<TRequest, TResponse>(
    IValidator<TRequest> validator) : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> HandleAsync(
        TRequest request,
        Func<CancellationToken, Task<TResponse>> next,
        CancellationToken cancellationToken)
    {
        ValidationResult result = await validator.ValidateAsync(request, cancellationToken);
        if (!result.IsValid) throw new ValidationException(result.Errors);
        return await next(cancellationToken);
    }
}
```

### Specification / policy object

Use to centralize business rules without burying them in controllers or EF queries.

```csharp
public interface ISpecification<T>
{
    Expression<Func<T, bool>> Criteria { get; }
}

public sealed class OrdersForTenantSpec(string tenantId) : ISpecification<Order>
{
    public Expression<Func<Order, bool>> Criteria => order => order.TenantId == tenantId;
}
```

Specifications help query reuse, but authorization should still be explicit and tested.

### Result pattern

Use when expected business outcomes should not be represented as exceptions.

```csharp
public sealed record Result(bool Succeeded, string Status)
{
    public static Result Success() => new(true, "success");
    public static Result NotFound() => new(false, "not_found");
    public static Result Forbidden() => new(false, "forbidden");
}
```

Avoid returning raw error strings to API callers when they may reveal internals.

### Options pattern

Use for validated configuration and fail-fast startup.

```csharp
public sealed class ToolPolicyOptions
{
    [Required]
    public string[] AllowedTools { get; init; } = [];
}

builder.Services.AddOptions<ToolPolicyOptions>()
    .Bind(builder.Configuration.GetSection("ToolPolicy"))
    .ValidateDataAnnotations()
    .Validate(options => options.AllowedTools.Length > 0, "At least one tool must be configured.")
    .ValidateOnStart();
```

### Outbox pattern

Use when database state and integration events must stay consistent.

```csharp
await using IDbContextTransaction transaction =
    await db.Database.BeginTransactionAsync(cancellationToken);

db.Orders.Add(order);
db.OutboxMessages.Add(new OutboxMessage
{
    Id = Guid.NewGuid(),
    Type = "OrderCreated",
    Payload = JsonSerializer.Serialize(new { order.Id }),
    OccurredUtc = DateTimeOffset.UtcNow
});

await db.SaveChangesAsync(cancellationToken);
await transaction.CommitAsync(cancellationToken);
```

Outbox dispatchers must be idempotent and should avoid putting secrets or unnecessary personal data in event payloads.

### Authorization handler

Use for resource-based authorization and BOLA prevention.

```csharp
public sealed class OrderOwnerRequirement : IAuthorizationRequirement
{
}

public sealed class OrderOwnerHandler : AuthorizationHandler<OrderOwnerRequirement, Order>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        OrderOwnerRequirement requirement,
        Order resource)
    {
        string? objectId = context.User.FindFirst("oid")?.Value;
        if (resource.OwnerObjectId == objectId || HasAppRole(context.User, "Orders.Admin"))
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }

    private static bool HasAppRole(ClaimsPrincipal user, string role) =>
        // Requires JWT bearer configuration with MapInboundClaims = false or RoleClaimType = "roles".
        user.FindAll("roles").Any(claim => claim.Value == role);
}
```

Register handlers in DI and write negative tests for unauthorized users and wrong tenants.

### Repository

Use repositories where they protect a meaningful boundary, such as aggregate persistence, test seams, or provider isolation. Do not create generic repositories that hide EF Core features and make security predicates harder to see.

```csharp
public interface IOrderRepository
{
    Task<Order?> GetForTenantAsync(Guid orderId, string tenantId, CancellationToken cancellationToken);
    Task SaveAsync(Order order, CancellationToken cancellationToken);
}
```

---

## Guidance

- prefer composition over inheritance
- keep authorization, validation, and persistence boundaries visible
- do not use patterns because a book says so
- start with simple code, add patterns where they remove real friction
- in ASP.NET Core, middleware, endpoint filters, DI, options, and authorization handlers already give you strong composition tools
- test the pattern boundary: authorization handler tests, validator tests, command handler tests, and integration tests for endpoint behavior

## Sources

- Microsoft Learn ASP.NET Core authorization and dependency injection guidance
- Microsoft Learn options pattern and configuration validation
- Microsoft Learn EF Core transactions and concurrency guidance
- OWASP API Security Top 10 2023 API1, API3, and API5
- OWASP ASVS v5.0.0 access control and architecture guidance
