# Data Classification and Storage Security

Use this file when the user stores sensitive data in SQL, Cosmos DB, Blob Storage, queues, caches, search/vector indexes, logs, or backups.

## Must implement

- Classify data by sensitivity and regulatory impact.
- Choose storage and access controls that match the classification.
- Encrypt in transit and at rest.
- Use private endpoints or restricted network paths where risk requires it.
- Apply least-privilege data-plane roles.
- Include tenant/user scope in storage layout and access checks.
- Define backup, restore, retention, and deletion behavior.

## Storage decision table

| Store | Key controls |
|---|---|
| Azure SQL | Entra auth, least-privilege users/roles, TDE, auditing, private endpoint where needed |
| Cosmos DB | partition key design, RBAC, private endpoint, consistency choice, backup policy |
| Blob Storage | container access private, managed identity, lifecycle policies, immutability where needed |
| Redis/cache | TTLs, no secrets unless approved, tenant-aware keys, private network |
| Search/vector index | tenant filters, document provenance, deletion sync, no unapproved sensitive data |
| Queues/topics | schema validation, no secrets, idempotency, dead-letter handling |

## Tenant-aware key example

```csharp
static string CacheKey(string tenantId, string resourceType, string resourceId) =>
    $"{tenantId}:{resourceType}:{resourceId}";
```

## Verification

- sensitive data is not stored in logs or telemetry
- tenant A cannot access tenant B storage objects
- backups and restore tests exist for critical stores
- deletion removes or excludes data from primary store, indexes, cache, and derived stores
- private endpoints/firewall rules match the deployment model

## Avoid

- using one shared storage key in application config
- storing personal data in vector indexes without retention/deletion plan
- exposing Blob containers publicly for convenience
- assuming encryption at rest replaces authorization

## Sources

- Microsoft Learn Azure SQL, Cosmos DB, Storage, Redis, AI Search, and Key Vault security guidance
- OWASP ASVS v5.0.0 data protection requirements
- GDPR/UK GDPR storage limitation and security principles
