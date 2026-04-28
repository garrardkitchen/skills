# Microsoft Agent Framework Workflows in .NET

Use this file when the user asks about multi-agent orchestration, handoff, evaluator patterns, or persisted sessions.

## Recommended workflow patterns

Agent workflow examples are conceptual unless they include persistence, authorization, rate limits, cancellation, telemetry, and data-protection controls. Do not promote model output to a write, send, execute, or publish operation without a deterministic gate.

### Handoff

Use handoff when a triage or coordinator agent should route work to a specialist.

```csharp
var workflow = AgentWorkflowBuilder.CreateHandoffBuilderWith(triageAgent)
    .WithHandoffs(triageAgent, [mathTutor, historyTutor])
    .WithHandoffs([mathTutor, historyTutor], triageAgent)
    .Build();
```

Best practices:

- make the routing criteria explicit
- keep specialist agents narrow in scope
- require the receiving agent to re-check authorization before taking action
- preserve tenant, user, session, and trace context across handoff
- treat handoff messages as untrusted input until schema and authorization checks pass

### Fan-out / fan-in

Use fan-out when parallel work is bounded and each child task has a clear purpose.

```csharp
Task<AgentResponse<TextResponse>> technicalTask =
    technicalAgent.RunAsync<TextResponse>($"Research technical aspects of {topic}");
Task<AgentResponse<TextResponse>> marketTask =
    marketAgent.RunAsync<TextResponse>($"Research market trends for {topic}");

await Task.WhenAll(technicalTask, marketTask);
```

Best practices:

- cap fan-out count
- add per-branch deadlines
- aggregate only validated outputs
- avoid letting one failed branch silently poison the whole result
- avoid unbounded `Task.WhenAll` over model/tool calls
- record per-branch cost, latency, and failure signals for abuse detection

### Evaluator pattern

Use an evaluator when one component must score or approve outputs before promotion.

```csharp
public sealed class EvaluatedResult<T>
{
    public required T Output { get; init; }
    public required bool Passed { get; init; }
    public required string Reason { get; init; }
}
```

Guidance:

- evaluators should use deterministic criteria where possible
- log evaluation metadata and safe summaries; redact secrets, personal data, prompts, and retrieved content where required
- use evaluators to gate publish/send/write operations
- keep evaluator decisions auditable and overrideable through a controlled approval path

## Session persistence

### Persist to disk

Disk persistence is suitable only for local experiments or controlled development. Production session state may contain prompts, retrieved content, personal data, credentials, or tool outputs and MUST be stored in an encrypted, access-controlled, retention-managed store.

```csharp
AgentSession session = await agent.CreateSessionAsync();
JsonElement serialized = agent.SerializeSession(session);

await File.WriteAllTextAsync(
    "session.json",
    serialized.ToString(),
    cancellationToken);
```

```csharp
JsonElement saved = JsonSerializer.Deserialize<JsonElement>(
    await File.ReadAllTextAsync("session.json", cancellationToken));

AgentSession resumed = await agent.DeserializeSessionAsync(saved);
```

### Persist to Azure Blob Storage

```csharp
using Azure.Identity;
using Azure.Core;
using Azure.Storage.Blobs;

string? managedIdentityClientId = builder.Configuration["Azure:ManagedIdentityClientId"];
TokenCredential credential = string.IsNullOrWhiteSpace(managedIdentityClientId)
    ? new ManagedIdentityCredential()
    : new ManagedIdentityCredential(managedIdentityClientId);

var blob = new BlobClient(
    new Uri("https://account.blob.core.windows.net/agent-sessions/session-001.json"),
    credential);

JsonElement serialized = agent.SerializeSession(session);
await blob.UploadAsync(
    BinaryData.FromString(serialized.ToString()),
    overwrite: true,
    cancellationToken);
```

Best practices:

- encrypt storage
- scope access with managed identity
- avoid storing secrets in session payloads
- expire or rotate stored sessions
- version session schemas if the app evolves
- record provenance for retrieved content and memory entries
- separate durable business state from resumable agent context
- do not trust restored session content as policy or developer instruction

## Workflow guidance

- handoff for specialization
- fan-out for bounded parallel work
- evaluator for promotion/approval
- persistent sessions for resumability, not as a substitute for durable business state
- kill switch for workflows that exceed policy, cost, latency, or tool-call thresholds
- negative tests for tool misuse, prompt injection, memory poisoning, and approval bypass

## Sources

- OWASP Top 10 for Agentic Applications
- Azure Agentic AI Security Baseline
- Microsoft Agent Framework documentation
- Microsoft Learn Azure Identity, Blob Storage, and OpenTelemetry guidance
