# NuGet, SBOM, and Build Security

Use this file when the user asks about package security, CI/CD, NuGet dependencies, software supply chain, SBOMs, signing, provenance, or build hardening.

## Must implement

- Use trusted package sources and review new dependencies.
- Pin or centrally manage package versions.
- Enable vulnerability scanning for direct and transitive packages.
- Generate SBOMs where the pipeline supports them.
- Use secret scanning and code scanning.
- Protect release branches and production deployment workflows.
- Avoid restoring packages from untrusted feeds.

## Central package management

```xml
<!-- Directory.Packages.props -->
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="Azure.Identity" Version="1.13.2" />
    <PackageVersion Include="Microsoft.EntityFrameworkCore.SqlServer" Version="9.0.0" />
  </ItemGroup>
</Project>
```

## Useful checks

```bash
dotnet list package --vulnerable --include-transitive
dotnet list package --outdated
dotnet nuget list source
```

## NuGet source mapping example

```xml
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <packageSourceMapping>
    <packageSource key="nuget.org">
      <package pattern="Microsoft.*" />
      <package pattern="System.*" />
      <package pattern="Azure.*" />
    </packageSource>
  </packageSourceMapping>
</configuration>
```

## Verification

- vulnerable package scan runs in CI
- secret scan runs before merge
- dependency update automation is reviewed
- SBOM is produced for releases
- production deploy requires approval
- package sources are pinned and reviewed

## Avoid

- floating versions in production libraries without a deliberate update process
- committing local NuGet credentials
- installing packages from unknown feeds for convenience
- ignoring transitive vulnerabilities

## Sources

- Microsoft Learn NuGet package management and package source mapping
- GitHub Advanced Security and Dependabot documentation
- OWASP Software Component Verification Standard and Dependency-Track concepts
- OWASP Top 10 2025 Software Supply Chain Failures
