# Secrets and Key Management in .NET

Use this file when the user asks about secrets, configuration safety, or Key Vault.

## Topics covered

- Azure Key Vault as a configuration source
- `SecretClient` usage
- `dotnet user-secrets` for local development
- secret rotation awareness
- ASP.NET Core Data Protection for protected app state
- managed identity and least-privilege access to secret stores

## Production rules

- Production secrets MUST NOT be stored in source code, checked-in config, container images, or plain pipeline variables.
- `dotnet user-secrets` is for local development only and is not encrypted as a production secret store.
- Prefer Azure Key Vault with managed identity or workload identity for deployed Azure applications.
- Never log secret values, access tokens, connection strings, Key Vault URIs with embedded credentials, or signed URLs.

## Local development with user secrets

```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:AppDb" "Server=(localdb)\\MSSQLLocalDB;Database=appdb;"
```

## Add Key Vault to configuration

```csharp
using Azure.Core;
using Azure.Identity;

string? managedIdentityClientId = builder.Configuration["Azure:ManagedIdentityClientId"];
TokenCredential credential = string.IsNullOrWhiteSpace(managedIdentityClientId)
    ? new ManagedIdentityCredential()
    : new ManagedIdentityCredential(managedIdentityClientId);

builder.Configuration.AddAzureKeyVault(
    new Uri(builder.Configuration["KeyVault:VaultUri"]!),
    credential);
```

Use `DefaultAzureCredential` only when the credential chain is intentional and documented. For deployed Azure workloads, prefer explicit managed identity selection.

## Use `SecretClient`

```csharp
using Azure.Core;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

string? managedIdentityClientId = builder.Configuration["Azure:ManagedIdentityClientId"];
TokenCredential credential = string.IsNullOrWhiteSpace(managedIdentityClientId)
    ? new ManagedIdentityCredential()
    : new ManagedIdentityCredential(managedIdentityClientId);

var client = new SecretClient(
    new Uri(builder.Configuration["KeyVault:VaultUri"]!),
    credential);

KeyVaultSecret secret = await client.GetSecretAsync("MyApiKey", cancellationToken: cancellationToken);
```

## Rotation guidance

- avoid caching secrets forever
- cache with bounded TTL
- reload on rotation windows where possible
- never log secret values
- design clients to tolerate secret/key rollover without downtime
- prefer short-lived tokens and managed identity over rotating long-lived shared secrets

## Data Protection

Use ASP.NET Core Data Protection for protecting trusted app state such as cookies, antiforgery tokens, and short-lived protected payloads. It is not a general-purpose replacement for Key Vault or long-term secret storage.

```csharp
builder.Services.AddDataProtection()
    .SetApplicationName("MyApp");
```

For multiple app instances, persist and protect the key ring with an approved store and key-protection mechanism, such as Azure Blob Storage plus Key Vault, so authentication cookies and protected payloads remain valid across deployments.

## Sources

- Microsoft Learn ASP.NET Core app secrets and Key Vault configuration
- Microsoft Learn Azure SDK authentication for .NET
- Microsoft Learn ASP.NET Core Data Protection
- OWASP Secrets Management and .NET Security Cheat Sheets
