# Testing Strategy in .NET

Use this file when the user asks about tests, naming, or test data patterns.

Adapt examples to the repository's chosen test framework. The important part is behavior coverage, clear names, and negative security cases; do not mix frameworks in one repo without a reason.

## Naming

Prefer behavior-oriented names:

- `CreateOrderAsync_ReturnsCreatedOrder_WhenInputIsValid`
- `DeleteDeviceAsync_ThrowsUnauthorizedAccessException_WhenUserLacksRole`

## Categories

With xUnit, use traits for coarse-grained categorization.

```csharp
[Trait("Category", "Unit")]
public class CreateOrderTests
{
}
```

## Data-driven tests

### `MemberData`

```csharp
public static IEnumerable<object[]> InvalidEmails()
{
    yield return ["bad"];
    yield return [""];
}

[Theory]
[MemberData(nameof(InvalidEmails))]
public void Rejects_Invalid_Email(string email)
{
    var action = () => EmailAddress.Parse(email);
    action.Should().Throw<FormatException>();
}
```

### `ClassData`

```csharp
public sealed class PriceCases : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return [1, 2, 3];
        yield return [2, 3, 5];
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

## Guidance

- unit tests for business logic
- integration tests for transport/database/auth boundaries
- use realistic object mothers/builders, not giant inline setup blocks
- prefer explicit test data over hidden magic
- use `Testcontainers` when mocks would hide database, broker, or cache behavior that matters
- test failure paths as carefully as success paths
- include authorization, tenant isolation, validation, concurrency, replay, and rate-limit cases where relevant
- use deterministic clocks, IDs, and fake external clients for reliable tests

## Security-oriented unit test example

```csharp
[Fact]
public async Task HandleAsync_ReturnsForbidden_WhenUserDoesNotOwnOrder()
{
    var user = TestUsers.Customer(objectId: "user-1");
    var order = new Order { Id = Guid.NewGuid(), OwnerObjectId = "user-2" };
    var repository = new FakeOrderRepository(order);
    var authorization = new FakeAuthorizationService(allowed: false);
    var handler = new CancelOrderHandler(repository, authorization);

    Result result = await handler.HandleAsync(
        new CancelOrderCommand(order.Id, "mistake", "idem-1"),
        user,
        CancellationToken.None);

    result.Status.Should().Be("forbidden");
}
```

## Minimum test checklist

- valid request succeeds
- invalid input is rejected
- unauthenticated request is rejected
- authenticated but unauthorized request is rejected
- wrong-owner or wrong-tenant resource is rejected
- concurrency conflict is handled
- duplicate/replayed message or webhook is safe
- secrets and internal exceptions are not returned to callers

## Sources

- Microsoft Learn .NET testing guidance
- Microsoft Learn ASP.NET Core integration testing
- OWASP ASVS v5.0.0 verification guidance
- OWASP API Security Top 10 2023
