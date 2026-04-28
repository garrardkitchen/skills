---
name: agentic-security-assessment
description: Assess Azure-hosted or Azure-integrated agentic AI repositories against the Azure Agentic AI Security Baseline and OWASP ASI01-ASI10, using OWASP Top Ten Web Application Security Risks 2025 codes A01:2025-A10:2025, OWASP API Security Top 10 2023, and ASVS v5.0.0 as cross-cutting lenses where web/API evidence affects agent risk. Use this skill whenever the user asks for agentic AI security assessment, OWASP agentic ASI01-ASI10 mapping, Azure agent security review, Terraform and code evidence review, agentic compliance gaps, or a security report with Mermaid architecture diagrams and implementation recommendations.
---

# Agentic Security Assessment Skill

## Overview

Use this skill to assess a repository against `docs/compliance/azure-agentic-ai-security-baseline.md`, specifically `ASI01` through `ASI10`.

This skill is designed for Azure-hosted or Azure-integrated agentic AI systems and MUST:

- read both Terraform and application code
- ignore `system-message.md` files as these are not enforceable controls
- extract evidence from real code and configuration
- map findings to `ASI01` through `ASI10`
- use OWASP Top Ten Web Application Security Risks 2025 and OWASP API Security Top 10 2023 as cross-cutting lenses when web/API vulnerabilities materially affect agent safety
- show where controls are addressed, partially addressed, or not addressed
- create a markdown assessment report
- generate a styled, colorized Mermaid diagram of the current implementation
- produce a gaps table with implementation guidance links relevant to the detected technology stack
- keep the report decision-ready for humans by leading with a compact executive snapshot, top concerns, and only then detailed evidence

For this skill, treat a system as agentic when it plans or reasons across multiple steps, invokes tools or external actions, persists memory or context, coordinates with other services or agents, or operates with partial autonomy before a human review point.

The default report location is:

`docs/compliance/assessments/`

---

## GPT-5.5 Output Principles

When generating the assessment, optimize for a human reader who needs to understand risk quickly:

- lead with the outcome and the top 3 concerns
- keep tables concise; move long evidence into detailed findings
- group repeated gaps instead of repeating the same recommendation under every ASI
- separate missing controls from missing evidence
- avoid speculative findings; every concern must cite code, Terraform, enforceable configuration, or a clearly stated absence of evidence
- use direct, balanced language suitable for security, engineering, and governance stakeholders

Use these concern markers consistently:

| Marker | Meaning | Use when |
|--------|---------|----------|
| 🔴 Critical concern | Immediate high-impact security gap | A production-impacting agentic risk is exploitable or a required control is plainly absent |
| 🟠 Material gap | Important implementation gap | A control is partial, weak, or missing for a significant workflow |
| 🟡 Evidence gap | Not proven from assessed scope | The control may exist elsewhere, but code/config evidence is missing |
| 🟢 Strength | Strong implementation evidence | The control is clearly implemented with enforceable evidence |

Also assign an evidence strength of `High`, `Medium`, or `Low`:

- `High`: direct Terraform, application code, or enforceable configuration proves the conclusion
- `Medium`: direct evidence exists but is incomplete, split across layers, or has unresolved scope limits
- `Low`: only secondary evidence exists, or the conclusion is mainly based on absence of evidence

---

## Before Starting — Ask These Questions

Before running the assessment, ask the user:

1. **Target repository**: Which repository or folder should be assessed?
2. **Assessment scope**: Whole repository, or only specific services / folders?
3. **Environment focus**: Production only, all environments, or a named environment?
4. **Output file name**: Use the default date-based report name, or a custom report name?
5. **Technology emphasis**: Are there known stacks to prioritize (for example `.NET`, `Python`, `Node.js`, `AKS`, `Terraform`, `Container Apps`, `Functions`, `Azure OpenAI`)?
6. **Evidence strictness**: Should the report only count code/config evidence, or also include supporting evidence from docs and CI/CD configuration?
7. **Report tone**: Use the default balanced tone, or prefer executive, engineering, or formal assurance wording?

If the user already provided enough detail, do not re-ask answered questions.

If the user does not express a report tone preference, use a balanced tone.

---

## Control Source

The canonical control source is:

`docs/compliance/azure-agentic-ai-security-baseline.md`

### Control source rules

1. First, look for that file in the target repository.
2. If it is missing, look for the same path in the current workspace, including the repository that contains this `SKILL.md`.
3. If it still cannot be found, stop and ask the user whether to:
   - provide the file path
   - copy the baseline into the target repository
   - continue with a reduced assessment using the skill's embedded expectations

Do not silently continue without a clear control source.

---

## What to Read

This skill MUST assess both infrastructure-as-code and application/runtime code.

### 1. Terraform and IaC

Read and assess, where present:

- `*.tf`
- `*.tfvars`
- `.terraform.lock.hcl`
- environment folders
- modules
- policy definitions
- `Dockerfile`, image build files, and workload runtime definitions when they define execution or dependency boundaries
- networking, identity, Key Vault, diagnostics, private endpoint, WAF, API gateway, monitoring, and execution environment configuration

Look specifically for evidence related to:

- identity and workload identity design
- RBAC and permission scoping
- managed identity role assignments, federation, and scope limits
- networking and private exposure controls
- logging and diagnostics
- Key Vault access policies, secret references, and encryption
- isolation of execution workloads
- API gateways and policy enforcement
- monitoring, kill switches, and containment patterns

### 2. Application code

Read and assess:

- application files such as `*.py`, `*.ts`, `*.tsx`, `*.js`, `*.jsx`, `*.cs`, `*.go`, and `*.java`
- prompt, instruction, template, and policy files such as `*.md`, `*.txt`, `*.json`, `*.yaml`, and similarly named files under `prompts/`, `instructions/`, `templates/`, or `policies/`
- dependency and package files such as `package.json`, lockfiles, `requirements.txt`, `pyproject.toml`, `poetry.lock`, `Pipfile`, `go.mod`, `go.sum`, `pom.xml`, `build.gradle`, and equivalent manifests
- API code
- orchestration code
- agent runtime code
- tool registration and invocation code
- memory and retrieval code
- approval workflows
- authentication and authorization middleware
- inter-agent communication code
- queue / workflow logic
- runtime limits, retries, and circuit breakers
- web/API boundary controls such as resource authorization, token validation, SSRF protection, upload handling, CORS, secure headers, production error handling, and audit logging

Look specifically for evidence related to:

- trusted vs untrusted prompt boundaries
- system prompt or instruction definitions and where they are loaded
- tool allowlists and parameter validation
- per-action authorization
- memory validation, provenance, and expiration
- approval UI or approval API logic
- workflow containment
- code execution safety
- inter-agent trust boundaries
- anomaly detection, kill switches, and quarantine logic

### 3. Secondary evidence

Use supporting evidence from the following only as secondary confirmation unless the user explicitly wants broader evidence:

- CI/CD workflows
- container definitions
- Helm charts
- docs
- README files
- architecture notes

Primary conclusions SHOULD be grounded in Terraform, application code, or directly enforceable configuration.

---

## Required Assessment Method

For each `ASI01` through `ASI10`:

1. Read the relevant control intent from `docs/compliance/azure-agentic-ai-security-baseline.md`
2. Search the repository for concrete evidence
3. Classify the result as one of:
   - `Addressed`
   - `Partially Addressed`
   - `Not Addressed`
   - `Not Applicable` (only when the architecture clearly lacks the relevant component)
4. Record:
   - exact file paths
   - key functions, resources, classes, modules, or configuration blocks
   - a short explanation of why the evidence maps to the ASI control
   - concern marker and evidence strength
5. If there is no evidence, say so explicitly

Do not over-credit intent. A TODO comment, design aspiration, or vague README note is not enough to mark a control as addressed.

Do not use "internal-only", "different repository", or "not yet implemented here" as justification for `Not Applicable`. If one evidence stream is missing, record the scope limitation and classify the uncovered control as `Not Addressed` or `Partially Addressed`, whichever is more accurate.

---

## Cross-Cutting OWASP Web/API Lens

Use OWASP Top Ten Web Application Security Risks 2025 and OWASP API Security Top 10 2023 to enrich ASI findings where web/API vulnerabilities change the agentic risk. Use the official Web Top Ten identifiers (`A01:2025` through `A10:2025`) whenever naming a web risk, especially beside remediation code. Do not create a second full OWASP matrix by default; that overwhelms the report and distracts from the ASI control source.

OWASP Web Top Ten 2025 identifiers:

| Code | Risk |
|------|------|
| `A01:2025` | Broken Access Control |
| `A02:2025` | Security Misconfiguration |
| `A03:2025` | Software Supply Chain Failures |
| `A04:2025` | Cryptographic Failures |
| `A05:2025` | Injection |
| `A06:2025` | Insecure Design |
| `A07:2025` | Authentication Failures |
| `A08:2025` | Software or Data Integrity Failures |
| `A09:2025` | Security Logging and Alerting Failures |
| `A10:2025` | Mishandling of Exceptional Conditions |

Instead, add a compact cross-cutting section only when evidence shows a material web/API concern, such as:

| Web/API concern | Typical ASI relationship | Evidence to look for |
|-----------------|--------------------------|----------------------|
| `A01:2025` Broken Access Control / API1:2023 BOLA / API5:2023 BFLA | `ASI02`, `ASI03`, `ASI10` | Missing resource checks, broad tool permissions, admin endpoints without policies |
| `A04:2025` Cryptographic Failures and secret exposure | `ASI03`, `ASI08` | Plaintext secrets, unmanaged keys, missing Key Vault references, weak token handling |
| `A05:2025` Injection | `ASI01`, `ASI02`, `ASI05` | Prompt-to-tool parameter flow, dynamic SQL, command execution, untrusted input reaching interpreters |
| `API7:2023` SSRF / unsafe outbound calls | `ASI01`, `ASI02`, `ASI05` | User-controlled URLs, arbitrary fetch tools, callback URLs, outbound redirects, private-network destinations |
| `A02:2025` Security Misconfiguration | `ASI03`, `ASI07`, `ASI09` | Public diagnostics, permissive CORS, unprotected Swagger, missing private endpoints |
| `A03:2025` Software Supply Chain Failures | `ASI04` | Unpinned dependencies, unreviewed MCP servers/plugins, missing lockfiles or SBOMs |
| `A09:2025` Security Logging and Alerting Failures | `ASI09`, `ASI10` | Missing audit logs for tool calls, approvals, rejected actions, model/tool anomalies |

When the assessed application uses `.NET` or `ASP.NET Core`, include short C# remediation snippets for the highest-value fixes. Keep snippets focused on one control each; do not turn the assessment into a tutorial.

### C# remediation examples

Use illustrative examples like these when they match the detected vulnerability and stack:

```csharp
// A01:2025 Broken Access Control / API1:2023 BOLA:
// authorize the caller against the specific resource.
app.MapGet("/api/orders/{id:guid}", async (
    Guid id,
    ClaimsPrincipal user,
    IAuthorizationService authorization,
    IOrderRepository orders,
    CancellationToken cancellationToken) =>
{
    Order? order = await orders.FindAsync(id, cancellationToken);
    if (order is null) return Results.NotFound();

    AuthorizationResult result = await authorization.AuthorizeAsync(user, order, "Orders.ResourceRead");
    return result.Succeeded ? Results.Ok(OrderDto.From(order)) : Results.Forbid();
})
.RequireAuthorization();
```

```csharp
// API7:2023 SSRF:
// illustrative validation gate. Pair this with egress firewall/proxy controls
// or a rebinding-resistant HTTP handler before sending the outbound request.
using System.Net;
using System.Net.Sockets;

static async Task<Uri> ValidateOutboundUriAsync(string candidate, CancellationToken cancellationToken)
{
    HashSet<string> allowedHosts = new(StringComparer.OrdinalIgnoreCase)
    {
        "api.contoso.com",
        "graph.microsoft.com"
    };

    if (!Uri.TryCreate(candidate, UriKind.Absolute, out Uri? uri))
        throw new InvalidOperationException("Invalid URL.");

    if (uri.Scheme != Uri.UriSchemeHttps || (!uri.IsDefaultPort && uri.Port != 443))
        throw new InvalidOperationException("Only HTTPS destinations on port 443 are allowed.");

    if (!allowedHosts.Contains(uri.Host))
        throw new InvalidOperationException("URL destination is not approved.");

    IPAddress[] addresses = await Dns.GetHostAddressesAsync(uri.IdnHost, cancellationToken);
    if (addresses.Length == 0 || addresses.Any(IsPrivateOrLoopback))
        throw new InvalidOperationException("Private or loopback destinations are not allowed.");

    return uri;
}

static bool IsPrivateOrLoopback(IPAddress address)
{
    if (IPAddress.IsLoopback(address))
        return true;

    if (address.AddressFamily == AddressFamily.InterNetwork)
    {
        byte[] bytes = address.GetAddressBytes();
        return bytes[0] == 10
            || bytes[0] == 127
            || (bytes[0] == 172 && bytes[1] >= 16 && bytes[1] <= 31)
            || (bytes[0] == 192 && bytes[1] == 168)
            || (bytes[0] == 169 && bytes[1] == 254);
    }

    if (address.AddressFamily == AddressFamily.InterNetworkV6)
    {
        byte[] bytes = address.GetAddressBytes();
        return address.IsIPv6LinkLocal || address.IsIPv6SiteLocal || (bytes[0] & 0xfe) == 0xfc;
    }

    return false;
}

builder.Services.AddHttpClient("approved-outbound")
    .ConfigurePrimaryHttpMessageHandler(() => new HttpClientHandler
    {
        AllowAutoRedirect = false
    });
```

DNS validation alone is not a complete SSRF defense because the connection can resolve the hostname again. Treat the code above as an application-layer gate and pair it with rebinding-resistant egress controls, such as an approved outbound proxy, firewall rules, or a handler that pins the validated destination.

```csharp
// A09:2025 Security Logging and Alerting Failures:
// audit rejected or high-risk tool calls without logging secrets or prompts verbatim.
logger.LogWarning(
    "Agent tool call rejected. TraceId={TraceId} Tool={ToolName} User={UserObjectId} Reason={Reason}",
    traceId,
    toolName,
    user.FindFirst("oid")?.Value,
    "Missing approval");
```

---

## ASI Evidence Expectations

Use these minimum expectations when assessing:

### ASI01 — Agent Behavior Hijacking

Look for:

- separation of trusted system instructions from user or retrieved input
- handling of untrusted content sources
- goal / plan change controls
- content filtering, prompt boundary logic, or plan drift detection
- for internet-facing agents, prompt abuse detection, rate limits, and kill-switch or containment hooks for hostile prompt patterns

### ASI02 — Tool Misuse

Look for:

- explicit tool registries or allowlists
- parameter schema validation
- destructive action gating
- dry-run or draft-only support
- tool-specific authorization or policy checks

### ASI03 — Identity & Privilege Abuse

Look for:

- managed identities
- short-lived credentials or federation
- separation of human and agent identities
- per-action authorization checks
- explicit scoping of tool/API permissions

### ASI04 — Agentic Supply Chain Vulnerabilities

Look for:

- third-party MCP or plugin controls
- pinned dependencies
- signed or verified component references
- model / prompt / connector inventory
- review gates for external components

### ASI05 — Unexpected Code Execution

Look for:

- sandboxing
- no direct execution of model output
- execution isolation
- validation before running generated code
- outbound network limits for execution environments

### ASI06 — Memory & Context Poisoning

Look for:

- memory segmentation
- validation of memory writes
- provenance metadata
- expiration / TTL / rollback of memory
- isolation by tenant, user, or session

### ASI07 — Insecure Inter-Agent Communication

Look for:

- mutual authentication or strong service identity
- encrypted channels
- schema validation for inter-agent messages
- controlled discovery / routing
- explicit trust boundaries

### ASI08 — Cascading Failures

Look for:

- retries with limits
- timeouts
- quotas
- circuit breakers
- step limits
- fan-out controls
- kill switches and staged containment

### ASI09 — Human-Agent Trust Exploitation

Look for:

- approval interfaces
- evidence-rich approval records
- step-up approval for high-risk actions
- anti-spoofing UX patterns
- honest presentation of uncertainty, scope expansion, and destructive side effects in approval flows
- immutable audit logs for approvals

### ASI10 — Rogue Agents

Look for:

- anomaly detection
- revocation or disablement procedures
- agent quarantine capability
- behavior baselines
- traceability of prompt, tool, model, and memory changes

---

## Output File

By default, create:

`docs/compliance/assessments/YYYY-MM-DD-agentic-security-assessment.md`

If the user provides a custom name, treat it as a file name unless they explicitly provide a relative path. Relative paths MUST be resolved from the target repository root.

Create the parent folder for the chosen output path if it does not exist. For the default path, create `docs/compliance/assessments/`.

---

## Required Report Structure

The report MUST include the following sections:

```markdown
# Agentic Security Assessment: [Repository or Service Name]

## Executive Snapshot

## Scope

## Assessed Architecture Overview

## Top Concerns

## ASI01–ASI10 Assessment Matrix

## Detailed Findings

### ASI01: ...
...
### ASI10: ...

## Current-State Mermaid Diagram

## Gaps and Recommendations

## Implementation Links

## Standards and References
```

Add `## Cross-Cutting Web/API Security Concerns` only when OWASP Top Ten Web Application Security Risks 2025 or OWASP API Security Top 10 2023 materially affects an ASI finding. Add `## C# Remediation Examples` only when `.NET`/`ASP.NET Core` is in scope and snippets will clarify the recommended fix.

### 1. Executive Snapshot

Include:

- overall assessment posture
- number of ASI controls addressed / partial / not addressed
- the single most important concern
- up to 3 highest-priority concerns
- whether the current implementation appears internet-facing, internal-only, or mixed
- a short reading guide that explains the concern markers

### 2. Scope

Include:

- repository path
- folders assessed
- Terraform scope
- application scope
- any excluded areas

### 3. Top Concerns

Before the full matrix, include a compact top-concerns table. Limit it to the top 3 unless there is a clear reason to show up to 5.

| Priority | ASI | Area | Why it matters | Evidence |
|----------|-----|------|----------------|----------|
| 🔴 Critical concern | ASI02 | Tool Misuse | Write-capable tools lack parameter policy and destructive-action approval | `src/agent/tools.py`, `infra/functions.tf` |

### 4. ASI01–ASI10 Assessment Matrix

Use a table like:

| ASI | Risk | Status | Concern | Evidence Strength | Terraform / Config Evidence | Application Evidence | Notes |
|-----|------|--------|---------|-------------------|-----------------------------|----------------------|-------|
| ASI01 | Agent Behavior Hijacking | Partially Addressed | 🟠 Material gap | Medium | `infra/apim.tf` | `src/agent/orchestrator.py` | Trusted instructions separated, but no plan-drift approval gate |

### 5. Detailed Findings

For each ASI section include:

- risk name
- status
- concern marker
- evidence strength
- where addressed
- exact evidence with file paths
- why the evidence counts
- what is missing

Keep each detailed finding skimmable:

- start with one sentence naming the concern or strength
- use no more than 3-5 evidence bullets unless the user asks for exhaustive evidence
- put detailed implementation links in the recommendations and references sections, not inside every finding

### 6. Cross-Cutting Web/API Security Concerns

Include this section only when OWASP Top Ten Web Application Security Risks 2025 or OWASP API Security Top 10 2023 materially affects an ASI finding.

Use a compact table with no more than 5 rows:

| Concern | OWASP lens | Related ASI | Evidence | Recommended fix |
|---------|------------|-------------|----------|-----------------|
| Missing resource authorization on tool-backed order endpoint | `A01:2025` Broken Access Control / `API1:2023` BOLA | ASI02, ASI03 | `src/Api/OrdersController.cs` | Add resource-based authorization before invoking the agent tool |

Do not duplicate every ASI finding here. Use this section to highlight web/API vulnerabilities that amplify agentic risk.

### 7. Current-State Mermaid Diagram

Create a Mermaid diagram that captures the current implementation.

The diagram MUST:

- be based on discovered code and infrastructure
- include users, Entra ID, apps, agent runtime, model endpoints, memory, tools, approval layers, logging, and external systems where present
- visually distinguish trusted boundaries, untrusted inputs, and sensitive components
- use color and styling
- use Mermaid-safe node identifiers and labels

Use Mermaid styling such as:

```mermaid
%%{init: {'theme':'base','themeVariables':{
  'primaryColor':'#0f172a',
  'primaryTextColor':'#e2e8f0',
  'primaryBorderColor':'#38bdf8',
  'lineColor':'#94a3b8',
  'secondaryColor':'#1e293b',
  'tertiaryColor':'#111827'
}}}%%
flowchart LR
  classDef user fill:#2563eb,stroke:#1d4ed8,color:#ffffff,stroke-width:2px;
  classDef agent fill:#7c3aed,stroke:#5b21b6,color:#ffffff,stroke-width:2px;
  classDef model fill:#f59e0b,stroke:#d97706,color:#ffffff,stroke-width:2px;
  classDef store fill:#059669,stroke:#047857,color:#ffffff,stroke-width:2px;
  classDef tool fill:#ea580c,stroke:#c2410c,color:#ffffff,stroke-width:2px;
  classDef guard fill:#e11d48,stroke:#9f1239,color:#ffffff,stroke-width:2px;
```

Use subgraphs, class definitions, icons in labels where helpful, and clear trust boundary labels. Build the diagram from discovered evidence: if Terraform shows Key Vault or private endpoints, add those nodes; if code shows tool registries, approval services, or model clients, add those nodes. Make it polished, but never invent components that were not found.

When writing Mermaid nodes, use safe identifiers with alphanumeric camelCase names and no spaces, such as `rolesNode`, `approvalApi`, or `agentRuntime1`.

When writing Mermaid labels:

- prefer quoted labels such as `rolesNode["App roles / Authorize attribute"]`
- do not place raw Mermaid delimiters inside labels, including `[`, `]`, `{`, `}`, `(`, and `)`
- do not paste code fragments like `[Authorize]`, JSON, policy snippets, or Markdown links directly into node text
- rewrite syntax-heavy labels into plain language, for example:
  - use `App roles / Authorize attribute` instead of `App roles / [Authorize]`
  - use `Prompt filtering and guardrails` instead of `Prompt filtering [APIM]`
  - use `Approval workflow` instead of `Approval(flow)`
- keep labels short; move detailed evidence into surrounding prose instead of the diagram

### 8. Gaps and Recommendations

This section MUST include a table for missing or partial controls:

| Priority | ASI | Gap | Impact | Evidence Layer | Relevant Technology | Recommended Implementation | Best-Practice Links |
|----------|-----|-----|--------|----------------|---------------------|----------------------------|---------------------|
| 🔴 Critical concern | ASI02 | No tool parameter validation | High | Application | Azure Functions + Python | Add JSON-schema validation and destructive-action approval gate | [Azure API Management policy docs](...), [OWASP guidance](...) |

Rules:

- include only real gaps, not hypothetical ones
- choose implementation links appropriate to the detected stack
- prefer official documentation for Azure, Terraform, and the app framework
- where useful, include OWASP, Microsoft, HashiCorp, language/framework docs, or vendor guidance
- show the recommended next change first; avoid long remediation essays in the table

### 9. C# Remediation Examples

Include this section when `.cs`, `.csproj`, ASP.NET Core, Azure Functions for .NET, Semantic Kernel, or Microsoft Agent Framework code is in scope and the highest-priority findings benefit from concrete implementation guidance.

Rules:

- include at most 3 snippets by default
- show defensive code only
- tailor examples to the actual finding, such as resource authorization, SSRF-safe outbound calls, upload limits, token validation, audit logging, or secure Azure credential selection
- label snippets as illustrative when repository-specific interfaces are invented
- do not include exploit payloads or runnable attack code

### 10. Implementation Links

Also include a short grouped link section by technology, such as:

- Azure identity
- Azure networking
- Terraform modules / policy
- application framework security
- OWASP agentic guidance
- OWASP Top Ten Web Application Security Risks 2025 (`A01:2025` through `A10:2025`)
- OWASP API Security Top 10 2023
- OWASP ASVS v5.0.0

### 11. Standards and References

End the report with concise citations to the official or authoritative sources used for the assessment.

Include references when relevant to the findings:

- Azure Agentic AI Security Baseline from the assessed repository or skill repository
- OWASP Top 10 for Agentic Applications
- OWASP Top Ten Web Application Security Risks 2025 (`A01:2025` through `A10:2025`)
- OWASP API Security Top 10 2023
- OWASP ASVS v5.0.0
- OWASP GenAI Security Project guidance
- Microsoft Learn guidance for Azure services found in the repository
- HashiCorp Terraform provider or module documentation for relevant IaC controls
- official language or framework security documentation for the detected application stack
- internal governance standards only when the user provides them or they are present in the repository

Rules:

- keep citations at the end so they support the assessment without interrupting the main narrative
- cite only sources that are relevant to the actual findings or recommendations
- prefer stable official documentation over blogs or generic articles
- do not imply formal compliance certification from citations alone

---

## Technology-Specific Link Rules

When recommending implementation guidance:

- prefer Microsoft Learn for Azure, Entra ID, API Management, Front Door, Key Vault, Azure Monitor, Defender for Cloud, Azure OpenAI, Azure AI Search, Functions, AKS, Container Apps, and App Service
- prefer HashiCorp docs for Terraform syntax, providers, and module practices
- prefer official framework docs for `.NET`, `Python`, `Node.js`, `Java`, `Go`, and frontend frameworks
- include OWASP Agentic / GenAI references where the recommendation is agent-specific
- include OWASP Top Ten Web Application Security Risks 2025, OWASP API Security Top 10 2023, and OWASP ASVS v5.0.0 when web/API controls materially affect the finding

Do not provide generic or random links if a clear technology-specific official source exists.

---

## Evidence Quality Rules

- Quote or cite file paths exactly.
- Keep evidence concise but specific.
- Prefer enforceable controls over comments or intention.
- Distinguish clearly between:
  - infrastructure-enforced controls
  - application-enforced controls
  - human or process controls

If a control is partly handled in Terraform but not enforced in application logic, mark it as `Partially Addressed`.

---

## Diagram Quality Rules

- No fake components.
- No hand-wavy labels like "security stuff".
- Use Mermaid-safe syntax:
  - node identifiers should be alphanumeric or camelCase without spaces
  - prefer quoted node labels
  - avoid raw square brackets or other Mermaid control characters inside label text
- Use color intentionally:
  - blue for users / entry points
  - purple for agent runtimes
  - green for storage / memory / logs
  - orange for tools / external systems
  - red / pink for guardrails, approval, or containment
- Label trust boundaries and approval points.
- Prefer one readable diagram over an oversized unreadable one.
- Before finalizing the report, do a quick syntax sanity check so the Mermaid block is parseable.

---

## Completion Criteria

This skill is complete only when it has:

1. read the baseline
2. assessed both Terraform and application code
3. produced an ASI01–ASI10 matrix
4. generated a markdown report
5. included a colored Mermaid diagram
6. documented real gaps in a recommendation table
7. attached implementation links appropriate to the detected technology
8. included concise standards and references citations
9. verified that the report includes Executive Snapshot, Scope, Architecture Overview, Top Concerns, ASI Matrix, Detailed Findings, Cross-Cutting Web/API Security Concerns when relevant, Mermaid Diagram, Gaps and Recommendations, C# Remediation Examples when relevant, Implementation Links, and Standards and References

If the repository is not actually agentic, say so clearly and produce a reduced report explaining why the ASI mapping is limited.
