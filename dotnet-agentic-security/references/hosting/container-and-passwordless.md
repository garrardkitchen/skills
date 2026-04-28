# Container Hardening and Passwordless Hosting

Use this file when the user asks about deployment posture.

## Container hardening

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS runtime
WORKDIR /app
COPY ./publish .
USER $APP_UID
ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080
ENTRYPOINT ["dotnet", "MyApp.Api.dll"]
```

## Passwordless Azure SQL

```csharp
string connectionString =
    "Server=tcp:myserver.database.windows.net,1433;" +
    "Database=mydb;" +
    "Authentication=Active Directory Default;" +
    "Encrypt=True;TrustServerCertificate=False;";

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

## Guidance

- run as non-root
- avoid baking secrets into images
- prefer managed identity or workload identity over client secrets
- pin base images deliberately and monitor them for vulnerabilities
- prefer minimal/chiseled images where compatible with diagnostics and globalization needs
- use read-only filesystems and dropped Linux capabilities where the platform supports them
- expose only the app port and avoid shipping build tools in runtime images
- generate SBOMs and scan images in CI/CD
- configure health/readiness endpoints without exposing sensitive dependency details

## Passwordless rules

- passwordless connection strings still require least-privilege database users or roles
- managed identity must be granted only the data-plane permissions the app needs
- local developer authentication must not imply production identity or RBAC is correctly configured
- integration tests should verify the deployed identity can connect without secrets

## Sources

- Microsoft Learn .NET container images and ASP.NET Core containers
- Microsoft Learn managed identities and Azure SQL Microsoft Entra authentication
- Microsoft Learn Azure SDK authentication for .NET
- OWASP Docker and Secrets Management guidance
