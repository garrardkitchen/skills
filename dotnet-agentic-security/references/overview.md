# Overview

This skill is a modular guidance system for modern `.NET` application development.

It is opinionated in favor of:

- secure-by-default design
- explicit architecture boundaries
- minimal privilege
- observable systems
- predictable deployment behavior
- clear test strategy

## Core themes

- OWASP Agentic Top 10 (`ASI01`-`ASI10`)
- OWASP Top 10 2025 mitigation guidance
- OWASP API Security Top 10 2023 mitigation guidance
- OWASP ASVS v5.0.0-aligned verification thinking for implementation completeness
- ASP.NET Core API best practices
- Microsoft Agent Framework orchestration patterns
- `Azure.Identity` and Azure-hosted security posture
- clean architecture and modular design
- high-signal operational guidance

## Design principles for the skill

1. Prefer modular loading over one huge guidance blob.
2. Prefer practical code over abstract policy.
3. Prefer production-credible examples over toy demos.
4. Treat identity, authorization, and observability as first-class design concerns.
5. Keep data access, agent orchestration, and transport boundaries explicit.

## Reference quality bar

Each reference file should help an engineer implement safely, not merely recognize a topic. When using or updating a reference, include:

- what must not be omitted for production
- what should be added based on threat model
- at least one practical C# example where implementation detail matters
- verification guidance, especially negative tests
- common anti-patterns to avoid
- official sources when the guidance is security-critical

## Recommended defaults

- use `async` end-to-end
- use `ProblemDetails` for API errors
- use policy-based authorization instead of ad hoc role string checks scattered everywhere
- use managed identity or workload identity in deployed Azure paths; use developer credentials only for local development
- use EF Core projections and bounded queries to avoid accidental over-fetching
- use OpenTelemetry early
- use typed options and input validation at the boundary

## Implementation priority

When guidance competes, prioritize in this order:

1. authorization and tenant isolation
2. identity, managed identity, and secret handling
3. boundary validation for API input, uploads, webhooks, files, and outbound HTTP
4. agent/tool controls, including prompt boundaries, tool allowlists, memory provenance, approvals, and kill switches
5. EF Core consistency, transactions, concurrency, caching, and privacy-sensitive data handling
6. rate limiting, resilience, health checks, OpenTelemetry, and audit evidence
7. security regression tests and supply-chain controls

Do not let lower-priority polish, such as project layout or documentation, distract from missing production security controls.

## Authoritative source families

Use official sources when giving security-critical guidance:

- OWASP Top 10 2025, OWASP API Security Top 10 2023, OWASP ASVS v5.0.0, and OWASP Cheat Sheet Series
- Microsoft Learn for ASP.NET Core security, authorization, CORS, antiforgery, Data Protection, rate limiting, and error handling
- Microsoft Learn for .NET secure coding and Azure SDK authentication
- Microsoft Learn for EF Core concurrency, transactions, performance, and provider behavior
- OpenTelemetry .NET documentation and semantic conventions
