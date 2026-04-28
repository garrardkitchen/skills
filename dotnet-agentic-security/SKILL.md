---
name: dotnet-agentic-security
description: Provide modular, security-prioritized guidance for building and reviewing secure .NET, ASP.NET Core, Azure-hosted, MCP, and agentic applications. Use for OWASP Agentic ASI01-ASI10, OWASP Top 10, OWASP API Security Top 10 2023, ASVS v5.0.0-aligned implementation guidance, identity, authorization, tenant isolation, secure APIs, EF Core, Azure.Identity, secrets, testing, observability, resilience, and production-ready C# examples.
---

# .NET Agentic Security Skill

## Overview

Use this skill when the user wants help creating, reviewing, or improving `.NET` or `ASP.NET Core` applications with strong engineering and security guidance.

This skill is optimized for:

- `ASP.NET Core` Web APIs
- background workers and hosted services
- class libraries
- agentic applications and MCP servers
- single-file and script-style C# utilities

This skill MUST:

- align agentic guidance to `ASI01` through `ASI10` using `docs/compliance/azure-agentic-ai-security-baseline.md`
- include OWASP web, API, and ASVS v5.0.0-aligned application security guidance with mitigation examples
- draw on Microsoft Learn guidance for ASP.NET Core security, .NET secure coding, Azure SDK authentication, EF Core, resilience, rate limiting, and OpenTelemetry
- prefer secure, production-appropriate patterns over demo-only shortcuts
- keep the main `SKILL.md` lean and load reference markdown files only when relevant
- include concrete sample code where the user asks for implementation guidance
- label examples as conceptual, minimal, or production-oriented when the distinction matters for safety

---

## When to Trigger

Use this skill when the user asks for help with topics such as:

- secure `.NET` application design
- secure `ASP.NET Core` APIs
- OWASP agentic security in C#
- OWASP web, API, or ASVS v5.0.0-aligned risk mitigation in C#
- Microsoft Agent Framework workflows
- MCP server security in `.NET`
- `Azure.Identity`, managed identity, MFA, RBAC, or app roles
- resource-level authorization, BOLA/BFLA, tenant isolation, or multitenancy
- clean architecture, SOLID, GoF patterns, or EDA in `.NET`
- EF Core, provider switching, or performance issues such as `N+1`
- OpenTelemetry, rate limiting, background workers, SSE, SignalR, WebSockets, gRPC, or test strategy
- Service Bus, webhooks, replay protection, correlation, containers, passwordless hosting, or secrets
- README or changelog guidance for `.NET` repositories

---

## Before Starting — Ask These Questions

Ask only the questions that matter to the user’s request.

1. What app shape is involved?
   - ASP.NET Core API
   - worker/background service
   - class library
   - agentic app
   - MCP server
   - single-file or script app
2. Is the goal:
   - new implementation
   - refactor
   - security review
   - architecture guidance
   - sample code only
3. Which topics are in scope?
   - OWASP agentic
   - OWASP web
   - APIs
   - identity
   - data
   - testing
   - observability
   - patterns/architecture
4. What deployment target matters?
   - local dev only
   - Azure-hosted
   - containerized
   - mixed local and Azure
5. Which data stores or transports are involved?
   - SQLite
   - Azure SQL
   - Cosmos DB
   - SSE
   - queues/events
6. Does the user want:
   - minimal examples
   - production-ready examples
   - clean architecture scaffolding
   - review checklists

Do not ask all questions by default. Ask only what is needed to narrow scope.

---

## Control Source

For agentic security guidance, use:

- `docs/compliance/azure-agentic-ai-security-baseline.md`

For web and API security guidance, use these authoritative sources conceptually and map them to practical `ASP.NET Core` mitigations:

- OWASP Top 10 2025
- OWASP API Security Top 10 2023
- OWASP Application Security Verification Standard (ASVS) v5.0.0
- OWASP Cheat Sheet Series, especially authorization, authentication, REST, .NET, secrets, and input validation
- Microsoft Learn for ASP.NET Core security, .NET secure coding, Azure SDK authentication, managed identities, EF Core, rate limiting, resilience, Data Protection, and OpenTelemetry

When the user asks for guidance rather than assessment, use the baseline and OWASP references to drive implementation recommendations, not audit scoring.

---

## Implementation Priority Ladder

When several concerns apply, prioritize implementation guidance in this order so engineers are not misled by lower-level polish before core controls:

1. **Authorization and tenant isolation** — policy checks, resource ownership, BOLA/BFLA prevention, cross-tenant negative tests.
2. **Identity and secrets** — Microsoft identity platform, managed identity/workload identity, MFA/Conditional Access at the identity layer, no production secrets in code or plain config.
3. **Boundary validation** — DTO allowlists, FluentValidation, file/upload constraints, SSRF-safe outbound calls, webhook signatures and replay windows.
4. **Agent/tool controls** — prompt/data trust boundaries, tool allowlists, per-tool authorization, downstream re-authorization, approvals, memory provenance, kill switches.
5. **Data consistency and privacy** — EF concurrency, transactions/outbox, cache TTLs, retention, redaction, personal-data minimization.
6. **Resilience and observability** — rate limits, timeouts, non-idempotent retry discipline, circuit breakers, audit trails, OpenTelemetry with PII controls.
7. **Security tests and supply chain** — authz regression tests, tenant isolation tests, replay tests, dependency scanning, SBOM, pinned/package-reviewed dependencies.

Call out `MUST` items when omitting them would create an insecure implementation. Use `SHOULD` for strong defaults that may vary by threat model.

---

## Progressive Loading Rules

Do not read every reference file up front.

Load only the files needed for the user’s request:

- always start with `references/overview.md`
- load `references/overview/topic-relationship-map.md` when the user asks how topics relate or needs a visual/conceptual map
- load `references/owasp-agentic/asi01-asi10.md` when the user asks about agentic risks, agent workflows, MCP servers, or agent orchestration
- load `references/owasp-web/top-10-web-risks.md` when the user asks about web app security, API security, validation, headers, authn/authz, SSRF, or logging
- load `references/api/http-api-best-practices.md` for HTTP API design, OpenAPI, versioning, validation, pagination, Problem Details, idempotency, or secure endpoint design
- load `references/api/http-hardening-and-errors.md` for middleware ordering, CORS, headers, CSRF/antiforgery, request limits, safe uploads, forwarded headers, and error handling
- load `references/agentic/microsoft-agent-framework-workflows.md` for handoff, fan-out/fan-in, evaluator patterns, workflow orchestration, or session persistence
- load `references/security/azure-identity-local-and-deployed.md` for `DefaultAzureCredential`, managed identities, local developer auth, or deployed Azure auth
- load `references/security/identity-mfa-rbac.md` for MFA, RBAC, app roles, claims, policy-based auth, and Microsoft identity platform integration
- load `references/security/authorization-and-access-control.md` for BOLA/BFLA, resource authorization, ownership checks, tenant-aware access control, privileged actions, and negative authz tests
- load `references/security/token-validation-and-claims.md` for JWT/OIDC validation, issuer/audience/lifetime checks, `scp` vs `roles`, app-only tokens, delegated tokens, and multi-tenant claims
- load `references/security/mcp-server-security.md` for MCP tool exposure, per-tool authorization, transport security, audit, and approval patterns
- load `references/security/secure-coding.md` for guard clauses, validation, injection prevention, configuration handling, and secure defaults
- load `references/security/secrets-and-key-management.md` for Key Vault, user secrets, Data Protection, secret rotation, or configuration secret safety
- load `references/security/cryptography-and-data-protection.md` for encryption, hashing, HMAC, random tokens, password hashing, ASP.NET Core Data Protection, and key rotation
- load `references/security/ssrf-and-safe-outbound-http.md` for SSRF prevention, callback URLs, remote fetches, safe outbound `HttpClient`, redirects, and private-network destination blocking
- load `references/security/file-upload-and-content-safety.md` for uploads, attachments, archive extraction, malware scanning, quarantine, and content-safety workflows
- load `references/security/privacy-data-retention-and-redaction.md` for personal data, retention, deletion, telemetry redaction, logs, exports, GDPR/UK GDPR-aware implementation, and AI data handling
- load `references/architecture/clean-architecture.md` for project layout and dependency boundaries
- load `references/architecture/solid-and-gof-patterns.md` for SOLID and Gang of Four patterns
- load `references/architecture/event-driven-architecture.md` for outbox, messaging, domain events, integration events, and eventually consistent workflows
- load `references/runtime/concurrency-and-background-work.md` for `SemaphoreSlim`, worker coordination, retry boundaries, and job safety
- load `references/runtime/sse-and-streaming.md` for SSE and streaming endpoints
- load `references/data/ef-core-and-provider-switching.md` for EF Core, code-first, provider switching, N+1 avoidance, and database selection
- load `references/data/ef-concurrency-and-caching.md` for concurrency tokens, `DbUpdateConcurrencyException`, transactions, ETags, distributed cache TTLs, and cache invalidation
- load `references/data/data-classification-and-storage-security.md` for data classification, sensitive storage, SQL/Cosmos/Blob/Search/cache/queue security, backup/restore, retention, and tenant-aware storage
- load `references/testing/unit-integration-testing.md` for test naming, categories, `Theory`, `MemberData`, `ClassData`, and data-loading patterns
- load `references/testing/security-integration-testing.md` for `WebApplicationFactory`, `Testcontainers`, authz regression tests, and infrastructure-backed security verification
- load `references/testing/security-test-catalog.md` for BOLA/BFLA, tenant isolation, SSRF, replay, upload, rate-limit, agentic, and supply-chain test coverage
- load `references/libraries/refit-fluentvalidation-spectre-cli.md` for typed REST clients with Refit, request validation with FluentValidation, and production-grade command-line apps with Spectre.Console.Cli
- load `references/operations/otel-rate-limiting-resilience.md` for OpenTelemetry, resilience, rate limiting, and operational controls
- load `references/operations/resilience-health-and-exporters.md` for HTTP resilience pipelines, health checks, readiness/liveness, OTLP exporters, and production hardening
- load `references/operations/audit-logging-and-incident-response.md` for audit event schema, privileged-action logging, alerts, revocation, kill switches, and incident response hooks
- load `references/messaging/service-bus-webhooks-and-correlation.md` for Azure Service Bus, queues, webhooks, replay protection, idempotency, and distributed correlation
- load `references/multitenancy/multi-tenant-and-data-isolation.md` for tenant isolation, row-level access, tenant-aware caches/storage/queues, and cross-tenant tests
- load `references/realtime/signalr-websockets-grpc-security.md` for SignalR, WebSockets, gRPC, bidirectional streams, message limits, and realtime authorization
- load `references/hosting/container-and-passwordless.md` for container hardening, non-root images, passwordless Azure SQL, managed identity, and hosting posture
- load `references/hosting/azure-hosting-security.md` for Azure App Service, Functions, Container Apps, AKS, private endpoints, WAF, diagnostics, deployment slots, and managed identity hosting posture
- load `references/frontend/blazor-and-browser-security.md` for Blazor, Razor Pages, MVC, cookies, antiforgery, CSP, browser token handling, BFF patterns, and browser-facing SignalR
- load `references/supply-chain/nuget-sbom-and-build-security.md` for NuGet source mapping, package scanning, SBOMs, signing/provenance, CI/CD protection, dependency automation, and release security
- load `references/agentic/rag-memory-and-vector-store-security.md` for RAG, vector search, long-term memory, memory poisoning, retrieval provenance, tenant-isolated indexes, and prompt-injection-through-content risks
- load `references/agentic/semantic-kernel-and-azure-ai.md` for Semantic Kernel, Azure OpenAI, Azure AI Foundry, function calling, structured outputs, content safety, and model/tool controls
- load `references/docs/readme-and-changelog.md` for repository documentation guidance
- load `references/app-shapes/single-file-and-scripts.md` for top-level programs, script execution, and small-tool patterns
- load `references/patterns/recipes.md` when the user asks for compact starter snippets, reusable building blocks, or copy-paste-ready patterns

If the user asks for broad end-to-end guidance, load a small set of relevant files rather than the whole tree.

---

## Default Working Style

When responding:

1. identify the app shape and security posture needed
2. load only the relevant reference files
3. apply the implementation priority ladder before giving lower-priority polish
4. explain the recommended design in practical terms
5. provide code that is idiomatic for modern `.NET`
6. prefer:
    - async APIs
    - explicit authorization
    - resource ownership and tenant checks
    - typed options
    - dependency injection
    - structured logging
    - OpenTelemetry-ready instrumentation
    - least privilege and managed identity
7. avoid:
    - hard-coded secrets
    - production `DefaultAzureCredential` usage without explicit principal selection
    - broad static singletons without lifecycle reasoning
    - blocking calls in async flows
    - insecure demo shortcuts in production examples
    - broad exception swallowing
    - examples that disable security controls without explaining when that is safe

---

## Output Expectations

Tailor the output to the user’s request. Good outputs include:

- secure starter code
- architectural recommendations
- topic-specific best-practice checklists
- code review comments
- modular implementation plans
- sample project structures
- README or changelog examples

Whenever useful, include:

- why the pattern is recommended
- what risk it mitigates
- what trade-offs it introduces
- what to avoid
- whether the pattern is mandatory for production or optional by threat model
- official source references when the recommendation is security-critical

For implementation guidance, prefer this compact structure:

1. **Must implement** — controls the developer should not omit for production.
2. **Should implement** — strong defaults that may vary by threat model.
3. **C# example** — minimal but safe enough not to mislead.
4. **Verification** — tests or checks proving the control works.
5. **Avoid** — common shortcuts that create security gaps.

---

## Section Map

The main guidance areas for this skill are:

1. overview and trigger rules
2. app-shape selection
3. OWASP agentic guidance
4. OWASP web guidance
5. HTTP API best practices
6. Microsoft Agent Framework workflow patterns
7. identity, MFA, RBAC, app roles, and `Azure.Identity`
8. authorization, token validation, secure coding, secrets, crypto, SSRF, uploads, privacy, and MCP server security
9. clean architecture, SOLID, GoF, and EDA
10. concurrency, background work, and SSE
11. realtime transports, queues, webhooks, and correlation
12. EF Core, provider switching, concurrency, caching, storage security, and performance
13. multitenancy, data isolation, and privacy-sensitive boundaries
14. testing strategy, data-driven tests, and security test catalog
15. observability, resilience, health checks, rate limiting, audit logging, and incident response
16. containers, Azure hosting, passwordless hosting, supply chain, README, changelog, frontend, and app-shape-specific guidance
17. RAG, memory, vector store, Semantic Kernel, Azure AI, and agentic implementation controls

---

## Completion Criteria

This skill is complete only when it:

1. provides actionable `.NET` guidance instead of generic advice
2. maps agentic guidance to `ASI01` through `ASI10` when relevant
3. includes OWASP web, API, and ASVS v5.0.0-aligned mitigation guidance when relevant
4. uses only the needed reference files
5. includes practical code examples for implementation-focused requests
6. prioritizes authorization, identity, boundary validation, agent/tool controls, data consistency, observability, and tests in that order when risk demands it
7. leaves room for future topic growth through new subfolders
