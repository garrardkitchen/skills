---
name: agentic-security-assessment
description: Assess a repository against the Azure Agentic AI Security Baseline and OWASP ASI01-ASI10 using Terraform and application code evidence, then produce a markdown report, colored Mermaid diagram, and implementation gap table.
---

# Agentic Security Assessment Skill

## Overview

Use this skill to assess a repository against `/Users/kitcheng/.claude/skills/docs/compliance/azure-agentic-ai-security-baseline.md/docs/compliance/azure-agentic-ai-security-baseline.md`, specifically `ASI01` through `ASI10`.

This skill is designed for Azure-hosted or Azure-integrated agentic AI systems and MUST:

- read both Terraform and application code
- ignore `system-message.md` files as these are not enforceable controls
- extract evidence from real code and configuration
- map findings to `ASI01` through `ASI10`
- show where controls are addressed, partially addressed, or not addressed
- create a markdown assessment report
- generate a styled, colorized Mermaid diagram of the current implementation
- produce a gaps table with implementation guidance links relevant to the detected technology stack

For this skill, treat a system as agentic when it plans or reasons across multiple steps, invokes tools or external actions, persists memory or context, coordinates with other services or agents, or operates with partial autonomy before a human review point.

The default report location is:

`docs/compliance/assessments/`

---

## Before Starting — Ask These Questions

Before running the assessment, ask the user:

1. **Target repository**: Which repository or folder should be assessed?
2. **Assessment scope**: Whole repository, or only specific services / folders?
3. **Environment focus**: Production only, all environments, or a named environment?
4. **Output file name**: Use the default date-based report name, or a custom report name?
5. **Technology emphasis**: Are there known stacks to prioritize (for example `.NET`, `Python`, `Node.js`, `AKS`, `Terraform`, `Container Apps`, `Functions`, `Azure OpenAI`)?
6. **Evidence strictness**: Should the report only count code/config evidence, or also include supporting evidence from docs and CI/CD configuration?

If the user already provided enough detail, do not re-ask answered questions.

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
5. If there is no evidence, say so explicitly

Do not over-credit intent. A TODO comment, design aspiration, or vague README note is not enough to mark a control as addressed.

Do not use "internal-only", "different repository", or "not yet implemented here" as justification for `Not Applicable`. If one evidence stream is missing, record the scope limitation and classify the uncovered control as `Not Addressed` or `Partially Addressed`, whichever is more accurate.

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

## Summary

## Scope

## Assessed Architecture Overview

## ASI01–ASI10 Assessment Matrix

## Detailed Findings

### ASI01: ...
...
### ASI10: ...

## Current-State Mermaid Diagram

## Gaps and Recommendations

## Implementation Links

## References
```

### 1. Summary

Include:

- overall assessment posture
- number of ASI controls addressed / partial / not addressed
- most critical risks
- whether the current implementation appears internet-facing, internal-only, or mixed

### 2. Scope

Include:

- repository path
- folders assessed
- Terraform scope
- application scope
- any excluded areas

### 3. ASI01–ASI10 Assessment Matrix

Use a table like:

| ASI | Risk | Status | Terraform / Config Evidence | Application Evidence | Notes |
|-----|------|--------|-----------------------------|----------------------|-------|
| ASI01 | Agent Behavior Hijacking | Partially Addressed | `infra/apim.tf` | `src/agent/orchestrator.py` | Trusted instructions separated, but no plan-drift approval gate |

### 4. Detailed Findings

For each ASI section include:

- risk name
- status
- where addressed
- exact evidence with file paths
- why the evidence counts
- what is missing

### 5. Current-State Mermaid Diagram

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

When writing Mermaid nodes, use safe identifiers such as `rolesNode`, `approvalApi`, or `agentRuntime1`.

When writing Mermaid labels:

- prefer quoted labels such as `rolesNode["App roles / Authorize attribute"]`
- do not place raw Mermaid delimiters inside labels, including `[`, `]`, `{`, `}`, `(`, and `)`, unless you intentionally use valid Mermaid shape syntax
- do not paste code fragments like `[Authorize]`, JSON, policy snippets, or Markdown links directly into node text
- rewrite syntax-heavy labels into plain language, for example:
  - use `App roles / Authorize attribute` instead of `App roles / [Authorize]`
  - use `Prompt filtering and guardrails` instead of `Prompt filtering [APIM]`
  - use `Approval workflow` instead of `Approval(flow)`
- keep labels short; move detailed evidence into surrounding prose instead of the diagram

### 6. Gaps and Recommendations

This section MUST include a table for missing or partial controls:

| ASI | Gap | Impact | Evidence Layer | Relevant Technology | Recommended Implementation | Best-Practice Links |
|-----|-----|--------|----------------|---------------------|----------------------------|---------------------|
| ASI02 | No tool parameter validation | High | Application | Azure Functions + Python | Add JSON-schema validation and destructive-action approval gate | [Azure API Management policy docs](...), [OWASP guidance](...) |

Rules:

- include only real gaps, not hypothetical ones
- choose implementation links appropriate to the detected stack
- prefer official documentation for Azure, Terraform, and the app framework
- where useful, include OWASP, Microsoft, HashiCorp, language/framework docs, or vendor guidance

### 7. Implementation Links

Also include a short grouped link section by technology, such as:

- Azure identity
- Azure networking
- Terraform modules / policy
- application framework security
- OWASP agentic guidance

---

## Technology-Specific Link Rules

When recommending implementation guidance:

- prefer Microsoft Learn for Azure, Entra ID, API Management, Front Door, Key Vault, Azure Monitor, Defender for Cloud, Azure OpenAI, Azure AI Search, Functions, AKS, Container Apps, and App Service
- prefer HashiCorp docs for Terraform syntax, providers, and module practices
- prefer official framework docs for `.NET`, `Python`, `Node.js`, `Java`, `Go`, and frontend frameworks
- include OWASP Agentic / GenAI references where the recommendation is agent-specific

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
8. verified that the report includes Summary, Scope, Architecture Overview, ASI Matrix, Detailed Findings, Mermaid Diagram, Gaps and Recommendations, Implementation Links, and References

If the repository is not actually agentic, say so clearly and produce a reduced report explaining why the ASI mapping is limited.
