# Security-Focused Integration Testing

Use this file when the user asks how to test auth, API security, or infrastructure boundaries.

## `WebApplicationFactory`

```csharp
public sealed class ApiFactory : WebApplicationFactory<Program>
{
}
```

If the app under test uses top-level statements, expose `Program` from the app project so the test assembly can reference it:

```csharp
public partial class Program
{
}
```

## Authentication regression test

```csharp
[Fact]
public async Task GetOrder_Rejects_Request_WhenTokenMissing()
{
    await using var factory = new ApiFactory();
    HttpClient client = factory.CreateClient();

    HttpResponseMessage response = await client.GetAsync("/api/orders/11111111-1111-1111-1111-111111111111");

    Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
}
```

For true cross-tenant authorization tests, create an authenticated principal for a different tenant and assert the specific forbidden behavior.

## `Testcontainers`

Install packages:

```bash
dotnet add tests/MyApp.Tests package Testcontainers.MsSql
dotnet add tests/MyApp.Tests package Microsoft.AspNetCore.Mvc.Testing
```

Create a reusable container fixture that starts and stops the dependency programmatically:

```csharp
using Testcontainers.MsSql;

public sealed class SqlServerFixture : IAsyncDisposable
{
    private readonly MsSqlContainer _container =
        new MsSqlBuilder()
            .WithImage("mcr.microsoft.com/mssql/server:2022-CU14-ubuntu-22.04")
            .Build();

    public string ConnectionString => _container.GetConnectionString();

    public Task StartAsync(CancellationToken cancellationToken = default) =>
        _container.StartAsync(cancellationToken);

    public ValueTask DisposeAsync() => _container.DisposeAsync();
}
```

Wire the running container into `WebApplicationFactory` so the app under test uses the real dependency:

```csharp
public sealed class ApiFactory : WebApplicationFactory<Program>
{
    private readonly string _connectionString;

    public ApiFactory(string connectionString)
    {
        _connectionString = connectionString;
    }

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureAppConfiguration((_, config) =>
        {
            config.AddInMemoryCollection(new Dictionary<string, string?>
            {
                ["ConnectionStrings:AppDb"] = _connectionString,
                ["Database:Provider"] = "SqlServer"
            });
        });
    }
}
```

Use the fixture from tests:

```csharp
[Fact]
public async Task GetOrder_ReturnsUnauthorized_WhenTokenMissing()
{
    await using var sql = new SqlServerFixture();
    await sql.StartAsync();

    await using var factory = new ApiFactory(sql.ConnectionString);
    HttpClient client = factory.CreateClient();

    HttpResponseMessage response = await client.GetAsync("/api/orders/11111111-1111-1111-1111-111111111111");

    Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
}
```

Useful programmatic patterns:

- call `StartAsync()` and `DisposeAsync()` from fixtures so tests fully control lifecycle
- use one container per test collection for speed, or one per test for stronger isolation
- wait for readiness before issuing requests when the service depends on migrations or seed data
- inject container connection strings through `WebApplicationFactory` configuration overrides
- keep test data setup explicit so container-backed tests remain reproducible

## Other container targets

```csharp
using Testcontainers.PostgreSql;
using Testcontainers.Redis;

var postgres = new PostgreSqlBuilder().WithImage("postgres:15.1").Build();
var redis = new RedisBuilder().WithImage("redis:7.0").Build();

await postgres.StartAsync();
await redis.StartAsync();
```

## Suggested security test areas

- authorization policy enforcement
- tenant isolation
- SSRF allowlist validation
- webhook signature and replay checks
- MCP tool authorization and destructive-action gating

For a broader list, also load `references/testing/security-test-catalog.md`.

## Must-have implementation tests

```csharp
[Fact]
public async Task RetireDevice_ReturnsForbidden_WhenUserLacksWriteScope()
{
    await using var factory = new ApiFactory();
    HttpClient client = factory.CreateClientWithToken(TestTokens.WithScopes("devices.read"));

    HttpResponseMessage response = await client.PostAsJsonAsync(
        "/api/v1/devices/11111111-1111-1111-1111-111111111111/retire",
        new { reason = "test" });

    response.StatusCode.Should().Be(HttpStatusCode.Forbidden);
}
```

Security integration tests should prove controls fail closed, not just that happy paths work.

## Sources

- Microsoft Learn ASP.NET Core integration testing
- Testcontainers for .NET
- OWASP ASVS v5.0.0 verification guidance
- OWASP API Security Top 10 2023
