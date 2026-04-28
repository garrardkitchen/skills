# Privacy, Data Retention, and Redaction

Use this file when the user handles personal data, telemetry, logs, prompts, model outputs, uploads, exports, data retention, deletion, or GDPR/UK GDPR-aware implementation.

## Must implement

- Classify personal and sensitive data processed by the app.
- Minimize collected, stored, logged, and exported data.
- Enforce retention and deletion rules in code or operations.
- Redact secrets and personal data from logs and telemetry.
- Protect subject-rights workflows from unauthorized access.
- Avoid sending sensitive data to AI/model/tool providers unless approved.

## Redaction example

```csharp
public static class LogRedactor
{
    public static string RedactEmail(string value)
    {
        int at = value.IndexOf('@');
        return at <= 1 ? "***" : $"{value[0]}***{value[at..]}";
    }
}
```

## Retention job sketch

```csharp
public sealed class RetentionWorker(AppDbContext db, TimeProvider timeProvider)
{
    public async Task DeleteExpiredExportsAsync(CancellationToken cancellationToken)
    {
        DateTimeOffset now = timeProvider.GetUtcNow();
        Export[] expired = await db.Exports
            .Where(x => x.ExpiresUtc <= now)
            .ToArrayAsync(cancellationToken);

        db.Exports.RemoveRange(expired);
        await db.SaveChangesAsync(cancellationToken);
    }
}
```

## Verification

- logs do not contain access tokens, cookies, connection strings, or raw personal data
- export links expire
- deletion workflow is authorized and audited
- retention job removes expired records
- telemetry sampling/redaction does not drop required audit events
- AI prompts and memory stores exclude prohibited data categories unless approved

## Avoid

- logging request/response bodies by default
- storing model prompts forever without retention rules
- using production personal data in lower environments
- exposing subject-rights endpoints without strong authorization

## Sources

- UK GDPR and GDPR principles including minimization, storage limitation, and security of processing
- Microsoft Learn logging and OpenTelemetry guidance
- OWASP Logging and Privacy guidance
