# Reusable .NET Recipes

Use this file when the user asks for compact implementation patterns rather than full architecture guidance.

## Result pattern

```csharp
public sealed record Result<T>(bool Succeeded, T? Value, string? ErrorCode)
{
    public static Result<T> Success(T value) => new(true, value, null);
    public static Result<T> Failure(string errorCode) => new(false, default, errorCode);
}
```

Use stable error codes internally; map them to `ProblemDetails` at the API boundary without leaking stack traces or sensitive implementation details.

## Typed options access

```csharp
public sealed class BlobOptions
{
    public string ContainerUri { get; init; } = string.Empty;
}
```

```csharp
builder.Services.AddOptions<BlobOptions>()
    .Bind(builder.Configuration.GetSection("Blob"))
    .Validate(o => Uri.IsWellFormedUriString(o.ContainerUri, UriKind.Absolute), "ContainerUri must be absolute.")
    .ValidateOnStart();
```

## Safe outbound HTTP client

```csharp
builder.Services.AddHttpClient<IMyApiClient, MyApiClient>(client =>
{
    client.Timeout = TimeSpan.FromSeconds(10);
    client.DefaultRequestHeaders.Add("User-Agent", "my-app");
})
.AddStandardResilienceHandler();
```

## Resource authorization helper

```csharp
public static async Task<IResult> ForbidUnlessAuthorizedAsync<TResource>(
    ClaimsPrincipal user,
    TResource resource,
    string policy,
    IAuthorizationService authorization,
    Func<IResult> whenAllowed)
{
    AuthorizationResult result = await authorization.AuthorizeAsync(user, resource, policy);
    return result.Succeeded ? whenAllowed() : Results.Forbid();
}
```

Prefer explicit resource authorization in handlers where possible; helpers should not hide which resource and policy are being checked.

## ProblemDetails mapper

```csharp
static IResult ToHttpResult<T>(Result<T> result) =>
    result.Succeeded
        ? Results.Ok(result.Value)
        : Results.Problem(statusCode: 400, title: result.ErrorCode);
```

Do not pass raw exception messages into `ProblemDetails`.

## Guidance

- recipes are starting points, not architecture by themselves
- prefer composable patterns over giant utility classes
- wire recipes into clear boundaries and tests
- add authorization, validation, telemetry, and cancellation when moving recipes into production

## Sources

- Microsoft Learn ASP.NET Core Minimal API responses and Problem Details
- Microsoft Learn .NET HTTP resilience
- OWASP API Security Top 10 2023
