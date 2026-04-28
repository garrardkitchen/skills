# RAG, Memory, and Vector Store Security

Use this file when the user builds retrieval-augmented generation, vector search, long-term memory, embeddings, knowledge stores, or document-grounded agents.

## Must implement

- Treat retrieved content as untrusted data, not instructions.
- Track provenance for chunks, documents, memory entries, and generated summaries.
- Enforce tenant/user/session isolation in indexes and memory stores.
- Validate and authorize memory writes.
- Add TTL, retention, and deletion paths for memory.
- Prevent prompt injection from retrieved documents changing tool or system policy.
- Log retrieval and memory use safely for audit and debugging.

## Memory entry model

```csharp
public sealed record AgentMemoryEntry(
    string TenantId,
    string UserId,
    string SessionId,
    string SourceDocumentId,
    string TrustLevel,
    string Content,
    DateTimeOffset ExpiresUtc);
```

## Retrieval filter pattern

```csharp
public sealed record RetrievalQuery(string TenantId, string UserId, string SearchText);

public interface IRetrievalService
{
    Task<IReadOnlyList<RetrievedChunk>> SearchAsync(
        RetrievalQuery query,
        CancellationToken cancellationToken);
}
```

Every retrieval query should include tenant scope and any required authorization filter. Do not rely only on client-side filtering after retrieval.

## Prompt boundary rule

Wrap retrieved content as evidence or context and explicitly tell the model it is untrusted. Do not concatenate retrieved content into system or developer instructions.

## Verification

- tenant A cannot retrieve tenant B chunks
- poisoned document cannot override tool policy
- memory write without provenance is rejected
- expired memory is not returned
- deleted source document is removed or excluded from retrieval
- citations refer to source documents and not fabricated paths

## Avoid

- one global vector index with no tenant filter
- storing user secrets or raw credentials in memory
- treating high-similarity retrieval as truth
- letting model-generated summaries overwrite source provenance

## Sources

- OWASP Top 10 for Agentic Applications ASI01 and ASI06
- OWASP Agentic AI Threats and Mitigations
- Azure Agentic AI Security Baseline
- Microsoft Azure AI Search and Azure OpenAI security guidance
