# MCP Server Security in .NET

Use this file when the user is building or exposing an MCP server.

## Security goals

- authenticate callers
- authorize per tool or capability
- validate parameters
- log tool use
- rate limit expensive or risky operations
- gate destructive actions
- isolate tenants, sessions, and downstream credentials
- redact tool outputs before returning them to the model or caller
- provide a kill switch for risky tools and compromised connectors

## Per-tool authorization pattern

```csharp
public sealed record ToolInvocationContext(
    ClaimsPrincipal Principal,
    string ToolName,
    JsonElement Arguments);

public interface IToolAuthorizationService
{
    Task EnsureAllowedAsync(ToolInvocationContext context, CancellationToken cancellationToken);
}
```

```csharp
public sealed class ToolAuthorizationService : IToolAuthorizationService
{
    public Task EnsureAllowedAsync(ToolInvocationContext context, CancellationToken cancellationToken)
    {
        if (context.ToolName == "delete-device" && !context.Principal.IsInRole("DeviceAdmin"))
            throw new UnauthorizedAccessException("Caller is not allowed to invoke delete-device.");

        return Task.CompletedTask;
    }
}
```

## Rules

- never expose every internal operation as a tool
- validate argument shape before invocation
- distinguish read-only and write-capable tools
- log who invoked what, with which target, and what happened
- use explicit approval for destructive tools
- require transport security and strong caller identity
- authorize both at the MCP boundary and again in the downstream service that performs the action
- scope credentials per tool or capability rather than sharing one broad service credential
- treat remote MCP servers and plugins as supply-chain dependencies requiring review
- do not return secrets, raw exception details, access tokens, or excessive records in tool output
- bind tool calls to tenant, user, session, and correlation identifiers

## Operational safeguards

- add per-tool rate limits
- add timeouts and cancellation
- disable or quarantine misbehaving tools quickly
- keep tool outputs bounded and scrub sensitive data where needed

## Audit fields

For production tool use, capture:

- caller subject and tenant
- tool name and version
- target resource identifier
- approval identifier when required
- normalized argument summary, excluding secrets
- result status and error category
- correlation or trace identifier

## Destructive tool gate

Write-capable tools SHOULD require:

1. parameter validation
2. per-tool authorization
3. downstream resource authorization
4. explicit approval for destructive or externally visible effects
5. idempotency or replay protection where retries can occur
6. audit record before and after execution

## Sources

- OWASP Top 10 for Agentic Applications
- OWASP API Security Top 10 2023
- OWASP ASVS v5.0.0 and Authorization Cheat Sheet
- Model Context Protocol security guidance where available
