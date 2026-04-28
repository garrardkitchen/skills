# Security Test Catalog for .NET

Use this file when the user asks what security tests to write or wants assurance that implementation controls work.

## API authorization tests

- missing token returns `401`
- valid token missing policy returns `403`
- user cannot access another user's object
- tenant A cannot access tenant B object
- delegated token cannot call app-only endpoint
- app-only token cannot call user-context endpoint unless explicitly allowed
- admin endpoint rejects normal user

## Boundary validation tests

- over-posted property is ignored or rejected
- invalid enum/string/date/number is rejected
- oversized body is rejected
- disallowed file extension is rejected
- path traversal filename is rejected
- SSRF URL to metadata/private/localhost destination is rejected

## Agentic tests

- retrieved prompt injection cannot change tool policy
- unapproved tool call is rejected
- destructive tool requires approval
- memory write without provenance is rejected
- expired memory is not returned
- runaway fan-out hits quota/kill switch

## Messaging and webhook tests

- invalid signature is rejected
- stale timestamp is rejected
- replayed message is rejected
- duplicate queue message is idempotent
- poison message is dead-lettered after bounded retries

## Example xUnit test

```csharp
[Fact]
public async Task GetOrder_ReturnsForbidden_WhenOrderBelongsToAnotherTenant()
{
    await using var factory = new ApiFactory();
    HttpClient client = factory.CreateClientWithToken(TestTokens.ForTenant("tenant-a"));

    HttpResponseMessage response = await client.GetAsync("/api/orders/order-from-tenant-b");

    response.StatusCode.Should().Be(HttpStatusCode.Forbidden);
}
```

## Pipeline checks

- `dotnet test`
- dependency vulnerability scan
- secret scan
- container/image scan where applicable
- IaC scan where applicable
- SBOM generation for releases

## Sources

- OWASP ASVS v5.0.0
- OWASP API Security Top 10 2023
- Microsoft Learn ASP.NET Core integration testing
- Microsoft Learn WebApplicationFactory
