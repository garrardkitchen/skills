# Secure Coding Patterns in .NET

Use this file for practical coding guidance.

## Guard clauses

```csharp
public static class Guard
{
    public static string AgainstNullOrWhiteSpace(string? value, string parameterName)
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new ArgumentException("Value is required.", parameterName);

        return value;
    }
}
```

## Options validation

```csharp
builder.Services.AddOptions<MyApiOptions>()
    .Bind(builder.Configuration.GetSection("MyApi"))
    .ValidateDataAnnotations()
    .Validate(o => Uri.IsWellFormedUriString(o.BaseUrl, UriKind.Absolute), "BaseUrl must be absolute.")
    .ValidateOnStart();
```

## Safer error handling

- return stable error contracts
- log server details internally
- avoid leaking stack traces to clients

## Avoid injection

- use EF Core or parameterized commands
- sanitize identifiers if you must build dynamic query fragments
- never concatenate untrusted input into SQL, shell commands, or URLs without strict policy

## Secure defaults

- bind only what you need
- disable unused endpoints
- prefer explicit serializers and request-size limits where risk justifies it

## High-risk coding areas

Treat these as first-class review targets:

- **Path traversal**: never combine untrusted path segments into file paths without normalization and base-directory enforcement.
- **Archive extraction**: protect against Zip Slip by validating every extracted path remains under the intended directory.
- **Deserialization**: do not deserialize untrusted payloads into arbitrary runtime types or enable unsafe polymorphic type resolution.
- **XML parsing**: disable DTD/external entity resolution unless there is a tightly controlled reason.
- **Regex/ReDoS**: use timeouts for complex regular expressions over untrusted input.
- **SSRF**: validate scheme, host, port, redirect behavior, and private-network destinations.
- **Shell/process execution**: prefer APIs over shell commands; if unavoidable, use fixed executable paths and allowlisted arguments.
- **Cryptography**: do not design custom cryptographic protocols; use framework or platform primitives.
- **Timing-sensitive comparisons**: use `CryptographicOperations.FixedTimeEquals` for signatures or MACs.
- **Logging**: redact tokens, secrets, cookies, authorization headers, personal data, and model/tool payloads where sensitive.

## Safe file path pattern

```csharp
static string ResolveUnderBaseDirectory(string baseDirectory, string untrustedName)
{
    string safeName = Path.GetFileName(untrustedName);
    string fullBase = Path.GetFullPath(baseDirectory);
    if (!fullBase.EndsWith(Path.DirectorySeparatorChar))
        fullBase += Path.DirectorySeparatorChar;

    string fullPath = Path.GetFullPath(Path.Combine(fullBase, safeName));

    if (!fullPath.StartsWith(fullBase, StringComparison.Ordinal))
        throw new InvalidOperationException("Path escapes the allowed directory.");

    return fullPath;
}
```

## Sources

- Microsoft .NET secure coding guidelines
- Microsoft ASP.NET Core security documentation
- OWASP .NET Security Cheat Sheet
- OWASP Input Validation, File Upload, SSRF, and Secrets Management Cheat Sheets
