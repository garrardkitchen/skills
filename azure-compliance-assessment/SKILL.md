---
name: azure-compliance-assessment
description: Assess or prove alignment against the Azure Compliance Baseline for ISO 27001, GDPR, UK GDPR, and EU AI Act using infrastructure and application evidence, then produce markdown tables and a styled Mermaid diagram.
---

# Azure Compliance Assessment Skill

## Overview

Use this skill to assess a repository against `Users/kitcheng/.claude/skills/docs/compliance/azure-compliance-baseline.md`.

This skill is designed for Azure-hosted or Azure-integrated services and MUST:

- read both Terraform/infrastructure configuration and application/runtime code
- use the Azure compliance baseline as the control source
- ask the user whether they want an `assessment` or `proof` output before starting
- extract evidence from real code and configuration
- map evidence to the baseline's control areas across identity, infrastructure, applications, privacy, and AI governance
- create a markdown report with a mode-specific compliance table
- generate a styled, colorized Mermaid diagram of where the controls are addressed in the solution

The default report location is:

`docs/compliance/assessments/`

---

## Before Starting — Ask These Questions

Before running the assessment, ask the user:

1. **Mode**: Do they want an `assessment` or a `proof`?
2. **Target repository**: Which repository or folder should be assessed?
3. **Assessment scope**: Whole repository, or only specific services / folders?
4. **Environment focus**: Production only, all environments, or a named environment?
5. **Exposure model**: Internet-facing, internal-only, or mixed?
6. **Output file name**: Use the default date-based report name, or a custom report name?
7. **Technology emphasis**: Are there known stacks to prioritize (for example `.NET`, `Python`, `Node.js`, `AKS`, `Terraform`, `Container Apps`, `Functions`, `App Service`, `Azure Front Door`, `API Management`)?
8. **Evidence strictness**: Should the report only count code/config evidence, or also include supporting evidence from docs and CI/CD configuration?

If the user already provided enough detail, do not re-ask answered questions.

If the user does not express a preference for mode, recommend `assessment`.

---

## Mode Rules

The skill MUST explicitly confirm one of the following modes before evaluating controls:

### 1. Assessment mode

Use this when the user wants a full compliance view, including passes and failures.

The main markdown table MUST:

- include assessed control rows even when evidence is missing
- use `✅` for controls that pass based on concrete evidence
- use `❌` for controls that are not met, are only partly evidenced, or cannot be evidenced from the assessed scope
- include short notes explaining why

Every `❌` row SHOULD say whether the issue is:

- evidence found but the control is not met
- not evidenced in the assessed scope

### 2. Proof mode

Use this when the user wants a proof-style output that only shows what the solution demonstrably meets.

The main markdown table MUST:

- include only controls with concrete evidence
- use `✅` for every included row
- omit controls that are not met, not evidenced, or out of scope
- stay evidence-led and avoid implied compliance for omitted controls

In proof mode, the summary MUST explicitly state that omitted controls are not proven by the assessed evidence.

---

## Control Source

The canonical control source is:

`docs/compliance/azure-compliance-baseline.md`

### Control source rules

1. First, look for that file in the target repository.
2. If it is missing, look for the same path in the current workspace, including the repository that contains this `SKILL.md`.
3. If it still cannot be found, stop and ask the user whether to:
   - provide the file path
   - copy the baseline into the target repository
   - pause until the canonical baseline is available

Do not silently continue without a clear control source.

---

## Baseline Domains to Cover

Assess the full baseline across these domains:

1. Identity and access baseline
2. Azure resource and infrastructure baseline
3. Application baseline
4. Privacy baseline for GDPR and UK GDPR
5. ISO 27001-specific requirements
6. GDPR-specific requirements
7. UK GDPR-specific requirements
8. EU AI Act baseline
9. Control expectations by service type
10. Minimum evidence the organisation should retain
11. Recommended implementation guardrails in Azure

When a domain is not relevant to the architecture, explain why clearly instead of guessing.

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
- landing-zone configuration
- identity, networking, Key Vault, monitoring, diagnostics, WAF, Front Door, Application Gateway, API Management, private endpoint, and compute/runtime configuration
- workload runtime definitions such as `Dockerfile`, container definitions, deployment manifests, and host/orchestration files where they define security boundaries

Look specifically for evidence related to:

- Microsoft Entra integration and workload identity design
- RBAC and role assignment scope
- app role or API authorization support
- network exposure, edge protection, and private access paths
- diagnostics, logging, monitoring, and alert routes
- Key Vault use, secret references, and encryption settings
- environment separation and landing-zone governance
- policy enforcement, tagging, allowed locations, and resource restrictions
- backup, recovery, retention, and resilience controls

### 2. Application code

Read and assess:

- application files such as `*.py`, `*.ts`, `*.tsx`, `*.js`, `*.jsx`, `*.cs`, `*.go`, and `*.java`
- configuration and policy files such as `*.json`, `*.yaml`, `*.yml`, `*.xml`, `.env.example`, and framework config files
- API code
- authentication and authorization middleware
- role or claims checks
- identity platform integration code
- logging, auditing, and telemetry hooks
- data-handling, retention, masking, and deletion logic
- consent, privacy, or user-notice flows
- AI feature toggles, model integration, transparency notices, and human-review workflows
- secure delivery and environment-specific configuration code

Look specifically for evidence related to:

- token validation, issuer/audience checks, and role/scope enforcement
- separation of Azure RBAC, Entra roles, Graph permissions, and application authorization
- secure input handling and boundary validation
- internet-facing protections such as stronger auth, abuse resistance, and session controls
- internal-only protections such as restricted exposure and trusted path assumptions
- privacy-by-design controls, minimisation, retention, subject rights support, and DPIA-related triggers
- AI transparency, human oversight, and risk controls where AI capabilities exist
- auditability and evidence retention

### 3. Secondary evidence

Use supporting evidence from the following only as secondary confirmation unless the user explicitly wants broader evidence:

- CI/CD workflows
- container definitions
- Helm charts
- docs
- README files
- architecture notes
- threat models
- runbooks

Primary conclusions SHOULD be grounded in Terraform, application code, or directly enforceable configuration.

---

## Required Assessment Method

For each relevant baseline control area:

1. Read the control intent from `docs/compliance/azure-compliance-baseline.md`
2. Search the repository for concrete evidence
3. Record whether the control is:
   - passed
   - failed
   - not proven from the assessed evidence
   - not relevant to the architecture
4. Record:
   - exact file paths
   - key functions, resources, classes, modules, or configuration blocks
   - whether the evidence is primarily infrastructure, application, or supporting evidence
   - a short explanation of why the evidence maps to the baseline
5. If there is no evidence, say so explicitly

Do not over-credit intent. A TODO comment, design aspiration, or vague README note is not enough to mark a control as met.

Do not use "internal-only", "different repository", or "not yet implemented here" as automatic proof of compliance. If evidence is missing, mark the item as failed in assessment mode or omit it in proof mode.

---

## Minimum Domain Expectations

Use these minimum expectations when assessing the full baseline:

### 1. Identity and access baseline

Look for:

- Microsoft Entra ID as the identity plane where Azure-native integration is available
- MFA and Conditional Access enforcement for human users
- workload identities constrained separately from user sign-ins
- separation of Entra roles, Azure RBAC, Graph permissions, and application roles
- explicit role, scope, or claim validation in the application
- access review, privileged access, and guest governance controls where relevant

### 2. Azure resource and infrastructure baseline

Look for:

- governed landing-zone patterns
- mandatory tagging and environment separation
- Azure Policy or equivalent guardrails
- approved edge services for internet-facing entry
- restricted administrative paths
- encryption, secret management, and key rotation support
- logging, monitoring, diagnostics, and alerting

### 3. Application baseline

Look for:

- secure application design and explicit authorization
- stronger internet-facing controls where exposure exists
- internal-only protections for trusted-path services
- secure configuration, dependency hygiene, and delivery protections
- API validation, error handling, and auditability

### 4. Privacy baseline for GDPR and UK GDPR

Look for:

- privacy-by-design implementation
- minimisation, purpose limitation, and retention controls
- support for subject rights workflows where relevant
- transfer, residency, telemetry, and support-boundary considerations
- data-protection logging and evidence of handling sensitive data carefully

### 5. ISO 27001-specific requirements

Look for:

- ownership, governance, review, and documented control operation
- asset, access, change, and incident control evidence
- monitoring and evidential records that support assurance

### 6. GDPR and UK GDPR-specific requirements

Look for:

- lawful-basis aware handling where applicable
- controller/processor boundary awareness
- security of processing
- breach-readiness, retention, and accountability support

### 7. EU AI Act baseline

Look for:

- AI feature identification and applicability analysis
- transparency notices for AI interaction where relevant
- human oversight, logging, and risk control points
- controls for prohibited or high-risk use cases where such features exist

### 8. Service-type expectations

Look for:

- stronger controls for internet-facing systems
- documented treatment of internal-only services without using that status to skip core controls
- alignment between the exposure model and actual implementation

### 9. Evidence retention expectations

Look for:

- audit logs
- configuration evidence
- role assignment evidence
- deployment or policy evidence
- operational records that support assurance

### 10. Minimum evidence the organisation should retain

Look for:

- architecture diagrams or data-flow diagrams where they are stored with the implementation
- asset inventory, service ownership, or accountability records referenced from the solution
- access-model evidence such as app roles, RBAC assignments, or Conditional Access design references
- vulnerability, hardening, logging, alerting, backup, or restore evidence linked to the deployed pattern
- privacy records, transfer records, or AI risk records where the solution processes personal data or uses AI
- exception records with approval, expiry, and remediation references where deviations are documented

### 11. Recommended implementation guardrails in Azure

Look for:

- management-group, subscription, and Azure Policy guardrails
- Entra group hygiene combined with separate application authorization
- managed identities or workload identity federation instead of long-lived shared secrets
- Key Vault, Defender for Cloud, Front Door or WAF, private endpoints, and central logging controls where applicable
- alignment between the baseline's recommended Azure guardrails and the actual deployed pattern

---

## Output File

By default, create:

`docs/compliance/assessments/YYYY-MM-DD-azure-compliance-assessment.md`

If the user provides a custom name, treat it as a file name unless they explicitly provide a relative path. Relative paths MUST be resolved from the target repository root.

Create the parent folder for the chosen output path if it does not exist. For the default path, create `docs/compliance/assessments/`.

---

## Required Report Structure

The report MUST include the following sections:

```markdown
# Azure Compliance Assessment: [Repository or Service Name]

## Summary

## Mode

## Scope

## Assessed Architecture Overview

## Compliance Table

## Detailed Findings

## Current-State Mermaid Diagram

## Gaps and Recommendations

## Implementation Links

## References
```

### 1. Summary

Include:

- overall posture
- which mode was used
- whether the implementation appears internet-facing, internal-only, or mixed
- most important strengths
- most important failures or missing proof points

### 2. Mode

State:

- whether the report is `assessment` or `proof`
- what the table semantics mean in that mode
- any caution about omitted rows in proof mode

### 3. Scope

Include:

- repository path
- folders assessed
- Terraform scope
- application scope
- any excluded areas
- whether supporting evidence was counted

### 4. Compliance Table

In assessment mode, use a table like:

| Domain | Control Area | Result | Infra Evidence | Application Evidence | Notes |
|--------|--------------|--------|----------------|----------------------|-------|
| Identity and access | MFA and Conditional Access for human users | ✅ | `infra/entra.tf` | `src/auth/config.ts` | Human sign-in protections are enforced through Entra and app validation |
| Application baseline | Explicit application authorization | ❌ | `infra/apim.tf` | `src/api/orders.ts` | API gateway exists, but app code does not enforce role checks on sensitive routes |

In proof mode, use a table like:

| Domain | Proven Control Area | Result | Infra Evidence | Application Evidence | Notes |
|--------|---------------------|--------|----------------|----------------------|-------|
| Identity and access | Explicit application authorization | ✅ | `infra/apim.tf` | `src/api/orders.ts` | Role checks are enforced in code and token validation is present |

Rules:

- in assessment mode, use only `✅` and `❌`
- in proof mode, use only `✅`
- do not include a failed row in proof mode
- keep control-area wording faithful to the baseline

### 5. Detailed Findings

For each domain include:

- the control area
- whether it passed, failed, or was not proven
- where it is addressed
- exact evidence with file paths
- why the evidence counts
- what is missing or still unproven

### 6. Current-State Mermaid Diagram

Create a Mermaid diagram that captures where the relevant compliance points are addressed across infrastructure and application components.

The diagram MUST:

- be based on discovered code and infrastructure
- show where identity, edge security, app authorization, secret handling, monitoring, privacy, and AI-governance-related controls appear where relevant
- visually distinguish infrastructure controls from application controls
- label trust boundaries, exposure boundaries, and sensitive components
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
  classDef entry fill:#2563eb,stroke:#1d4ed8,color:#ffffff,stroke-width:2px;
  classDef infra fill:#0f766e,stroke:#115e59,color:#ffffff,stroke-width:2px;
  classDef app fill:#7c3aed,stroke:#5b21b6,color:#ffffff,stroke-width:2px;
  classDef store fill:#059669,stroke:#047857,color:#ffffff,stroke-width:2px;
  classDef guard fill:#e11d48,stroke:#9f1239,color:#ffffff,stroke-width:2px;
  classDef identity fill:#f59e0b,stroke:#d97706,color:#ffffff,stroke-width:2px;
```

Use subgraphs, class definitions, icons in labels where helpful, and clear trust boundary labels. Build the diagram from discovered evidence: if Terraform shows Front Door, WAF, Key Vault, private endpoints, policy, or diagnostics, add those nodes; if code shows token validation, role checks, privacy workflows, logging middleware, or AI transparency handlers, add those nodes. Make it polished, but never invent components that were not found.

When writing Mermaid nodes, use safe identifiers with alphanumeric camelCase names and no spaces, such as `entraIdentity`, `frontDoorEdge`, or `privacyWorkflow1`.

When writing Mermaid labels:

- prefer quoted labels such as `roleChecks["App role checks"]`
- do not place raw Mermaid delimiters inside labels, including `[`, `]`, `{`, `}`, `(`, and `)`
- do not paste code fragments, JSON, policy snippets, or Markdown links directly into node text
- rewrite syntax-heavy labels into plain language
- keep labels short; move detailed evidence into surrounding prose instead of the diagram

### 7. Gaps and Recommendations

This section MUST include a table for failed or unproven control areas:

| Domain | Gap | Impact | Evidence Layer | Recommended Implementation | Best-Practice Links |
|--------|-----|--------|----------------|----------------------------|---------------------|
| Identity and access | No application role checks on sensitive API routes | High | Application | Add explicit role and scope checks in the API and align them with Entra app roles | [Microsoft Learn](...), [Framework auth docs](...) |

Rules:

- include only real gaps, not hypothetical ones
- do not present omitted proof-mode rows as if they are compliant
- choose implementation links appropriate to the detected stack
- prefer official documentation for Azure, Terraform, and the app framework

### 8. Implementation Links

Also include a short grouped link section by technology, such as:

- Microsoft Entra ID and identity platform
- Azure networking and edge protection
- Azure Key Vault and secrets
- logging and monitoring
- application framework security
- privacy and AI governance guidance

---

## Technology-Specific Link Rules

When recommending implementation guidance:

- prefer Microsoft Learn for Azure, Entra ID, API Management, Front Door, Application Gateway, Key Vault, Azure Monitor, Defender for Cloud, Azure OpenAI, Azure AI Search, Functions, AKS, Container Apps, and App Service
- prefer HashiCorp docs for Terraform syntax, providers, and module practices
- prefer official framework docs for `.NET`, `Python`, `Node.js`, `Java`, `Go`, and frontend frameworks
- prefer authoritative privacy or regulatory guidance when the recommendation is about GDPR, UK GDPR, or EU AI Act controls

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
- If a control is only partly evidenced, treat it as failed in assessment mode and omit it in proof mode unless the proven part alone fully satisfies the control wording.

---

## Diagram Quality Rules

- No fake components.
- No hand-wavy labels like "security stuff".
- Use Mermaid-safe syntax:
  - node identifiers should be alphanumeric or camelCase without spaces
  - prefer quoted node labels
  - avoid raw square brackets or other Mermaid control characters inside label text
- Use color intentionally:
  - blue for entry points
  - teal for infrastructure controls
  - purple for application components
  - green for data stores, secrets, and telemetry
  - red / pink for guardrails and compliance control points
  - amber for identity and access controls
- Label trust boundaries and approval points where relevant.
- Prefer one readable diagram over an oversized unreadable one.
- Before finalizing the report, do a quick syntax sanity check so the Mermaid block is parseable:
  - verify node identifiers are alphanumeric or camelCase with no spaces
  - verify labels use quoted strings
  - verify label text does not contain raw bracket-like control characters
  - verify subgraphs and node declarations are closed correctly

---

## Completion Criteria

This skill is complete only when it has:

1. asked the user whether they want `assessment` or `proof`
2. read the baseline
3. assessed both Terraform and application code
4. produced the correct mode-specific markdown table
5. generated a markdown report
6. included a colored Mermaid diagram
7. documented real gaps and recommendations
8. attached implementation links appropriate to the detected technology
9. verified that the report includes Summary, Mode, Scope, Architecture Overview, Compliance Table, Detailed Findings, Mermaid Diagram, Gaps and Recommendations, Implementation Links, and References
