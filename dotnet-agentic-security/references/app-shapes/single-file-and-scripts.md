# Single-File Apps and Scripts in C#

Use this file when the user wants lightweight utilities.

## Single-file console app with top-level statements

```csharp
using System.Net.Http.Json;

HttpClient client = new();
var result = await client.GetFromJsonAsync<object>("https://example.com/api/health");
Console.WriteLine(result);
```

Run it inside a small project with:

```bash
dotnet run
```

## Script-style execution with `dotnet-script`

```csharp
#!/usr/bin/env dotnet-script
Console.WriteLine("Hello from a script.");
```

Run with:

```bash
dotnet script hello.csx
```

## Guidance

- use single-file apps for utilities, migrations helpers, diagnostics, and automation
- keep production services in proper projects, not ad hoc scripts
- for scripts, still avoid hard-coded secrets and add logging when they touch real systems
- treat command-line arguments, file paths, URLs, and environment variables as untrusted input
- require `--dry-run` for destructive administrative tools
- use managed identity or developer credentials instead of embedding secrets
- add cancellation, timeouts, and structured exit codes for automation
- pin script dependencies and review supply-chain impact before using script runners in shared environments

## Safer utility skeleton

```csharp
using System.CommandLine;
using Microsoft.Extensions.Logging;

var pathOption = new Option<FileInfo>("--path") { IsRequired = true };
var dryRunOption = new Option<bool>("--dry-run", () => true);

var command = new RootCommand("Safe maintenance utility")
{
    pathOption,
    dryRunOption
};

command.SetHandler((file, dryRun) =>
{
    if (!file.Exists)
    {
        Console.Error.WriteLine("Input file does not exist.");
        Environment.ExitCode = 2;
        return;
    }

    Console.WriteLine(dryRun
        ? $"Would process {file.FullName}"
        : $"Processing {file.FullName}");
}, pathOption, dryRunOption);

return await command.InvokeAsync(args);
```

## Avoid

- running generated scripts against production without review
- storing production credentials in `.csx`, shell history, or environment variables on shared machines
- using scripts as long-lived services
- downloading and executing remote code as part of a convenience script

## Sources

- Microsoft .NET secure coding guidelines
- Microsoft Learn .NET CLI and worker service guidance
- OWASP Secrets Management and Software Supply Chain guidance
