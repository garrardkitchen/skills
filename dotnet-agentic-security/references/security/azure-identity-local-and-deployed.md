# Azure.Identity for Local and Deployed Environments

Use this file when the user asks about `DefaultAzureCredential`, local development, managed identities, or Azure-hosted auth patterns.

## Recommended mental model

- local development should use developer credentials
- deployed environments should use managed identity or workload identity
- application code should not need different business logic for each environment
- production code should make the intended deployed principal explicit
- broad credential chains are convenient locally but can hide wrong-principal and missing-RBAC mistakes

## Environment-aware credential selection

```csharp
using Azure.Identity;
using Azure.Core;

TokenCredential CreateCredential(IHostEnvironment environment, IConfiguration configuration)
{
    if (environment.IsDevelopment())
    {
        return new ChainedTokenCredential(
            new AzureCliCredential(),
            new VisualStudioCredential(),
            new VisualStudioCodeCredential());
    }

    string? managedIdentityClientId = configuration["Azure:ManagedIdentityClientId"];

    if (!string.IsNullOrWhiteSpace(managedIdentityClientId))
    {
        return new ManagedIdentityCredential(managedIdentityClientId);
    }

    return new ManagedIdentityCredential();
}
```

Use `DefaultAzureCredential` deliberately when you want its full chain and have documented which credentials are allowed:

```csharp
var credential = new DefaultAzureCredential(new DefaultAzureCredentialOptions
{
    ExcludeInteractiveBrowserCredential = true,
    ManagedIdentityClientId = builder.Configuration["Azure:ManagedIdentityClientId"]
});
```

## Production rule

Deployed Azure workloads SHOULD use managed identity or workload identity rather than client secrets. If user-assigned managed identity is used, configure the client ID explicitly so the app cannot silently select a different identity.

## Guidance

- prefer token-based authentication over connection strings or account keys where Azure SDKs support it
- prefer `ManagedIdentityCredential` or workload identity in deployed Azure paths
- use explicit local developer credential chains when repeatability matters
- document which local credential paths are allowed:
  - Azure CLI
  - Visual Studio
  - Visual Studio Code
- do not depend on interactive browser auth in unattended production paths
- assign least-privilege Azure RBAC roles to the workload identity and test those assignments in integration environments
- do not let local credential success hide missing production RBAC assignments

## Using Blob Storage

```csharp
var credential = CreateCredential(app.Environment, app.Configuration);
var blobClient = new BlobClient(
    new Uri(app.Configuration["Storage:BlobUri"]!),
    credential);
```

## What to avoid

- secrets in source code
- long-lived shared service principal secrets as the default production path
- production `DefaultAzureCredential` usage without documenting and constraining the expected credential source
- broad roles such as Owner or Contributor when a data-plane role is sufficient

## Sources

- Microsoft Learn Azure SDK authentication for .NET
- Microsoft Learn managed identity guidance
- Microsoft Learn ASP.NET Core secure authentication flows
