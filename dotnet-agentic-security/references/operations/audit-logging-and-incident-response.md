# Audit Logging and Incident Response

Use this file when the user asks about audit trails, security monitoring, incident response, kill switches, revocation, break-glass, or operational evidence.

## Must implement

- Audit authentication, authorization failures, privileged actions, data exports, admin changes, tool invocations, approvals, and secret/key access where applicable.
- Include correlation ID, subject, tenant, action, target, outcome, and timestamp.
- Redact secrets and sensitive payloads.
- Route security-relevant logs to monitored storage.
- Define alert conditions for abuse, anomalous access, and control failure.
- Provide revocation and kill-switch paths for high-risk agents/tools/integrations.

## Audit event model

```csharp
public sealed record AuditEvent(
    string CorrelationId,
    string TenantId,
    string SubjectId,
    string Action,
    string TargetType,
    string TargetId,
    string Outcome,
    DateTimeOffset OccurredUtc);
```

## Logging example

```csharp
logger.LogInformation(
    "Audit {Action} {TargetType} {TargetId} {Outcome} for {SubjectId} in {TenantId}",
    audit.Action,
    audit.TargetType,
    audit.TargetId,
    audit.Outcome,
    audit.SubjectId,
    audit.TenantId);
```

## Incident response hooks

- disable user, app, agent, or tool
- revoke credentials or managed identity role assignment
- rotate affected secrets
- quarantine queue/tool/workflow
- preserve audit evidence
- notify owner/security channel

## Verification

- privileged actions emit audit events
- failed authorization emits security signal without leaking data
- audit logs cannot be modified by normal app users
- kill switch blocks the affected tool or workflow
- alerts trigger on repeated failures, replay, tenant boundary failures, and runaway agent calls

## Sources

- Microsoft Learn logging, Azure Monitor, Application Insights, and Sentinel guidance
- OWASP Logging Cheat Sheet
- OWASP ASVS v5.0.0 logging and error handling requirements
- Azure Agentic AI Security Baseline
