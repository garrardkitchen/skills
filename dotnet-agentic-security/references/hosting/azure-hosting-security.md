# Azure Hosting Security for .NET

Use this file when the user deploys `.NET` apps to Azure App Service, Functions, Container Apps, AKS, Azure SQL, Storage, Service Bus, Key Vault, Azure OpenAI, or related services.

## Must implement

- Managed identity or workload identity for Azure resource access.
- Key Vault or approved secret store for secrets.
- Private endpoints or restricted network paths for data stores where risk requires it.
- WAF or approved edge protection for internet-facing apps.
- HTTPS-only and current TLS.
- Diagnostic settings, logs, metrics, and alerts.
- Least-privilege Azure RBAC and data-plane permissions.
- Environment separation for production and non-production.

## App Service / Functions checklist

- enable managed identity
- disable FTP/basic publishing where not needed
- set HTTPS-only
- use deployment slots deliberately
- configure health checks
- restrict SCM/Kudu access
- store app settings without production secrets where possible

## Container Apps / AKS checklist

- run as non-root
- configure ingress deliberately
- use workload identity
- apply resource limits
- restrict egress where feasible
- use readiness/liveness probes
- scan images and pin base images

## Passwordless Azure SQL example

```csharp
string connectionString =
    "Server=tcp:server.database.windows.net,1433;" +
    "Database=appdb;" +
    "Authentication=Active Directory Default;" +
    "Encrypt=True;TrustServerCertificate=False;";
```

## Verification

- deployed identity can access only required resources
- app fails safely when RBAC is missing
- public data endpoints are not exposed directly
- diagnostics reach Log Analytics/Application Insights
- secrets are absent from appsettings, source, images, and logs

## Sources

- Microsoft Learn App Service, Functions, Container Apps, AKS, Key Vault, Azure SQL, managed identity, and Azure Monitor guidance
- Azure Well-Architected Framework security pillar
- OWASP ASVS v5.0.0 deployment and configuration guidance
