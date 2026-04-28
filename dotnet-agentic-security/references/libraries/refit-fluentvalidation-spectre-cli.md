# Refit, FluentValidation, and Spectre.Console.Cli

Use this file when the user asks about common .NET productivity libraries for HTTP clients, validation, or CLI apps.

## Refit

Install packages:

```bash
dotnet add src/MyApp package Refit.HttpClientFactory
```

Define a typed API contract:

```csharp
using Refit;

public interface IWeatherApi
{
    [Get("/weather/{city}")]
    Task<WeatherResponse> GetWeatherAsync(string city, CancellationToken cancellationToken);
}
```

Register it with `HttpClientFactory`:

```csharp
builder.Services
    .AddRefitClient<IWeatherApi>()
    .ConfigureHttpClient(client =>
    {
        client.BaseAddress = new Uri("https://api.contoso.com/");
        client.Timeout = TimeSpan.FromSeconds(10);
    });
```

Guidance:

- keep Refit interfaces narrow and task-focused
- combine Refit with resilience handlers for retries and timeouts
- do not hide authorization or correlation concerns inside magic static clients
- attach authentication, correlation IDs, and user-agent headers through `HttpClientFactory` handlers
- validate and constrain any user-controlled path/query values before calling downstream APIs
- treat downstream API responses as untrusted input

```csharp
public sealed class CorrelationHandler(IHttpContextAccessor accessor) : DelegatingHandler
{
    protected override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
    {
        string correlationId = accessor.HttpContext?.TraceIdentifier ?? Guid.NewGuid().ToString("n");
        request.Headers.TryAddWithoutValidation("X-Correlation-Id", correlationId);
        return base.SendAsync(request, cancellationToken);
    }
}
```

## FluentValidation

Install packages:

```bash
dotnet add src/MyApp package FluentValidation.DependencyInjectionExtensions
```

Create a validator:

```csharp
using FluentValidation;

public sealed class OrderLineValidator : AbstractValidator<CreateOrderLineRequest>
{
    public OrderLineValidator()
    {
        RuleFor(x => x.Sku).NotEmpty();
        RuleFor(x => x.Quantity).GreaterThan(0);
    }
}

public sealed class CreateOrderRequestValidator : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderRequestValidator()
    {
        RuleFor(x => x.CustomerId).NotEmpty();
        RuleFor(x => x.Total).GreaterThan(0);
        RuleForEach(x => x.Lines).SetValidator(new OrderLineValidator());
    }
}
```

Register validators:

```csharp
builder.Services.AddValidatorsFromAssemblyContaining<CreateOrderRequestValidator>();
```

Use the validator at the boundary:

```csharp
app.MapPost("/api/orders", async (
    CreateOrderRequest request,
    IValidator<CreateOrderRequest> validator,
    CancellationToken cancellationToken) =>
{
    ValidationResult result = await validator.ValidateAsync(request, cancellationToken);
    if (!result.IsValid)
        return Results.ValidationProblem(result.ToDictionary());

    return Results.Accepted();
});
```

If your FluentValidation package or integration path does not expose `result.ToDictionary()`, replace it with a small compatibility helper:

```csharp
public static class ValidationResultExtensions
{
    public static IDictionary<string, string[]> ToValidationDictionary(this ValidationResult result) =>
        result.Errors
            .GroupBy(x => x.PropertyName)
            .ToDictionary(
                group => group.Key,
                group => group.Select(x => x.ErrorMessage).ToArray());
}
```

Guidance:

- validate at transport boundaries, not halfway through business logic
- keep validators deterministic and side-effect free
- prefer clear rule messages over opaque generic failures
- validation does not replace authorization, ownership checks, tenant checks, or business invariants
- reject over-posted fields with narrow DTOs rather than accepting entity-shaped request models

## Spectre.Console.Cli

Install packages:

```bash
dotnet add src/MyTool package Spectre.Console.Cli
dotnet add src/MyTool package Spectre.Console
```

Create command settings:

```csharp
public sealed class ImportSettings : CommandSettings
{
    [CommandOption("--path <PATH>")]
    public string Path { get; init; } = string.Empty;

    [CommandOption("--dry-run")]
    public bool DryRun { get; init; }
}
```

Create a command. Use `AsyncCommand<TSettings>` when the command performs async or I/O-bound work:

```csharp
public sealed class ImportCommand : AsyncCommand<ImportSettings>
{
    protected override async Task<int> ExecuteAsync(
        CommandContext context,
        ImportSettings settings,
        CancellationToken cancellationToken)
    {
        AnsiConsole.MarkupLine($"[green]Importing[/] from {Markup.Escape(settings.Path)}");
        await Task.Delay(10, cancellationToken);
        return 0;
    }
}
```

If your installed Spectre version uses a different override signature, follow the package version you reference in your project.

Register the command:

```csharp
var app = new CommandApp();
app.Configure(config =>
{
    config.AddCommand<ImportCommand>("import");
});

return await app.RunAsync(args);
```

Guidance:

- treat CLI arguments as untrusted input and validate them
- keep command handlers thin; delegate business logic into services
- use Spectre for structured output, not as a replacement for application architecture
- default destructive commands to dry-run unless explicitly confirmed
- avoid printing secrets, tokens, connection strings, or personal data
- return meaningful process exit codes for automation

## Sources

- Refit documentation
- FluentValidation documentation
- Spectre.Console.Cli documentation
- Microsoft Learn HttpClientFactory and .NET resilience guidance
- OWASP API Security Top 10 2023 API10
