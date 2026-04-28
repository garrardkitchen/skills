# File Upload and Content Safety

Use this file when the user handles uploads, generated files, attachments, archive extraction, image processing, import jobs, or model-ingested documents.

## Must implement

- Authenticate and authorize upload, download, and delete operations.
- Enforce server-side request and multipart limits.
- Allowlist file extensions and content types.
- Verify file signatures or magic numbers for formats where the type matters.
- Store files outside the web root.
- Generate trusted storage names; never trust the original filename.
- Quarantine and scan untrusted files where risk requires it.
- Validate archive extraction paths to prevent Zip Slip.
- Set archive entry count, size, and compression-ratio limits to reduce archive-bomb risk.
- Treat uploaded content as untrusted even after storage.

## Upload endpoint pattern

This API-oriented example assumes bearer-token or non-browser clients. If the endpoint is called from a cookie-authenticated browser flow, use antiforgery services and middleware instead of disabling antiforgery.

```csharp
app.MapPost("/api/files", async (
    IFormFile file,
    ClaimsPrincipal user,
    IWebHostEnvironment environment,
    CancellationToken cancellationToken) =>
{
    if (file.Length is 0 or > 10 * 1024 * 1024)
        return Results.BadRequest("Invalid file size.");

    string extension = Path.GetExtension(file.FileName);
    string[] allowed = [".pdf", ".txt", ".csv"];
    if (!allowed.Contains(extension, StringComparer.OrdinalIgnoreCase))
        return Results.BadRequest("File type is not allowed.");

    string storageName = $"{Guid.NewGuid():n}{extension}";
    string directory = Path.Combine(environment.ContentRootPath, "quarantine");
    Directory.CreateDirectory(directory);

    string path = Path.Combine(directory, storageName);
    await using FileStream stream = new(path, FileMode.CreateNew);
    await file.CopyToAsync(stream, cancellationToken);

    return Results.Accepted(new { storageName });
})
.RequireAuthorization("Files.Upload")
.DisableAntiforgery();
```

## Safe archive extraction check

```csharp
static string ResolveArchiveEntry(string destinationDirectory, string entryName)
{
    string destination = Path.GetFullPath(destinationDirectory);
    if (!destination.EndsWith(Path.DirectorySeparatorChar))
        destination += Path.DirectorySeparatorChar;

    string target = Path.GetFullPath(Path.Combine(destination, entryName));

    if (!target.StartsWith(destination, StringComparison.Ordinal))
        throw new InvalidOperationException("Archive entry escapes destination.");

    return target;
}
```

## Verification

- unauthenticated upload is rejected
- unauthorized tenant cannot read another tenant's file
- cookie-authenticated browser uploads require antiforgery tokens
- oversized file is rejected
- disallowed extension is rejected
- path traversal filename cannot escape storage directory
- archive entry with `../` is rejected
- archive entries cannot escape via sibling path prefixes such as `/safe/outside`
- archive bombs, excessive file counts, and unexpected file signatures are rejected
- uploaded content is not served from executable paths

## Avoid

- `.DisableAntiforgery()` on cookie/browser upload forms.
- Saving files under `wwwroot`.
- Trusting `ContentType`, extension, or original filename alone.
- Extracting archives without entry count, total extracted size, per-file size, and compression-ratio limits.
- Following symlinks, hardlinks, device files, or other special archive entries during extraction.
- Running complex parsers for untrusted files in the main application process when parser compromise is in scope.
- Passing untrusted documents directly into prompts without RAG/content-safety controls.

## Sources

- Microsoft Learn ASP.NET Core file uploads and antiforgery
- OWASP File Upload Cheat Sheet
- OWASP API Security Top 10 2023 API4
