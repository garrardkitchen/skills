# Azure Agentic AI Security Baseline

## Status

Draft baseline standard

## Contents

- [Purpose](#purpose)
- [Governance and adoption](#governance-and-adoption)
- [Scope](#scope)
- [What this document treats as an agentic application](#what-this-document-treats-as-an-agentic-application)
- [Key security principles](#key-security-principles)
- [Applicability model](#applicability-model)
  - [Internet-facing agentic systems](#internet-facing-agentic-systems)
  - [Internal-only agentic systems](#internal-only-agentic-systems)
- [Agentic system architecture baseline](#agentic-system-architecture-baseline)
- [Identity and access baseline](#identity-and-access-baseline)
  - [Microsoft Entra ID and workload identity](#microsoft-entra-id-and-workload-identity)
  - [Agent identities as non-human identities](#agent-identities-as-non-human-identities)
  - [App roles, RBAC, and tool authorization](#app-roles-rbac-and-tool-authorization)
- [Azure infrastructure baseline for agentic AI](#azure-infrastructure-baseline-for-agentic-ai)
  - [Governance and landing zones](#governance-and-landing-zones)
  - [Network architecture](#network-architecture)
  - [Compute and runtime isolation](#compute-and-runtime-isolation)
  - [Secrets and cryptography](#secrets-and-cryptography)
  - [Logging, monitoring, and detection](#logging-monitoring-and-detection)
- [Application baseline for agentic AI](#application-baseline-for-agentic-ai)
  - [Prompt and instruction boundary design](#prompt-and-instruction-boundary-design)
  - [Tooling model](#tooling-model)
  - [Human approval model](#human-approval-model)
  - [Memory and retrieval model](#memory-and-retrieval-model)
- [OWASP Top 10 for Agentic Applications mapping](#owasp-top-10-for-agentic-applications-mapping)
  - [ASI01: Agent Behavior Hijacking](#asi01-agent-behavior-hijacking)
  - [ASI02: Tool Misuse](#asi02-tool-misuse)
  - [ASI03: Identity & Privilege Abuse](#asi03-identity-privilege-abuse)
  - [ASI04: Agentic Supply Chain Vulnerabilities](#asi04-agentic-supply-chain-vulnerabilities)
  - [ASI05: Unexpected Code Execution](#asi05-unexpected-code-execution)
  - [ASI06: Memory & Context Poisoning](#asi06-memory-context-poisoning)
  - [ASI07: Insecure Inter-Agent Communication](#asi07-insecure-inter-agent-communication)
  - [ASI08: Cascading Failures](#asi08-cascading-failures)
  - [ASI09: Human-Agent Trust Exploitation](#asi09-human-agent-trust-exploitation)
  - [ASI10: Rogue Agents](#asi10-rogue-agents)
- [Control expectations by deployment type](#control-expectations-by-deployment-type)
- [Azure implementation guardrails](#azure-implementation-guardrails)
- [Minimum evidence the organisation should retain](#minimum-evidence-the-organisation-should-retain)
- [Non-compliance and exceptions](#non-compliance-and-exceptions)
- [Summary](#summary)
- [References](#references)

## Purpose

This document defines a minimum security baseline for agentic AI systems deployed on Azure, including:

- internet-facing agentic applications
- internal-only agentic applications
- single-agent and multi-agent systems
- agents that plan, call tools, use memory, retrieve data, execute workflows, or delegate tasks

It is written as a reusable Azure baseline focused on:

- Azure infrastructure and platform controls
- Microsoft Entra ID and the Microsoft identity platform
- application and API controls for agentic systems
- the OWASP Top 10 for Agentic Applications (2026)

This document is a technical and operational baseline intended to support secure design, implementation, and assurance. It should be reviewed by security, privacy, legal, and architecture stakeholders before being adopted as a binding organisational standard.

## Governance and adoption

This document is a reference baseline until formally adopted by the organisation's governance process.

Before this document is used as a binding standard, the organisation SHOULD define:

- the approving authority for the baseline
- the control owners accountable for implementation
- the exception and waiver process
- the review cadence
- the relationship between this baseline and legal, privacy, records-management, and AI governance processes

Once adopted, the organisation MAY use the mandatory language in this document as internal policy language. Until then, it SHOULD be treated as a template or reference standard.

## Scope

This baseline applies to Azure-hosted or Azure-integrated agentic AI systems including:

- Azure OpenAI, Azure AI Foundry, Azure AI Search, and related AI services
- custom agents using LLMs, tools, memory stores, vector databases, or orchestration frameworks
- APIs, web applications, background workers, Functions, Logic Apps, Container Apps, AKS, App Service, and VMs used by agentic systems
- MCP-compatible integrations, plugins, connectors, webhooks, and third-party tool endpoints
- human approval interfaces, operator consoles, and administrator workflows

## What this document treats as an agentic application

For the purposes of this baseline, an agentic application is an AI-enabled system that does one or more of the following:

- plans or reasons over multiple steps
- selects or invokes tools
- takes actions in external systems
- persists context or memory across steps or sessions
- coordinates with other agents or services
- acts with partial autonomy before a human reviews the result

## Key security principles

- Agents are non-human actors with real operational impact and MUST be treated accordingly.
- Least privilege must apply to agent identity, agent memory, agent tools, and downstream systems.
- The security boundary is not just the model endpoint; it includes prompts, memory, retrieved content, tool calls, approvals, credentials, logs, and operators.
- Internet-facing agentic systems require stricter containment, abuse controls, and observability than internal-only deployments.
- Human approval is a control surface and can itself be exploited; approval UX must be designed securely.
- Tool combinations matter as much as individual tools.

## Applicability model

### Internet-facing agentic systems

Internet-facing agentic systems MUST implement heightened controls for:

- prompt and content abuse resistance
- tool abuse prevention
- rate limiting and runaway execution containment
- tenant and session isolation
- telemetry, alerting, and incident response
- human approval hardening

### Internal-only agentic systems

Internal-only agentic systems MUST still implement the full baseline, but MAY rely more heavily on:

- private networking
- managed device conditions
- trusted operator paths
- segmented internal APIs and private endpoints

Internal-only status MUST NOT be used to omit:

- strong identity controls
- tool authorization
- memory security
- audit logging
- secrets protection
- kill switches and containment controls

## Agentic system architecture baseline

Every production agentic system SHOULD be described in an architecture model that identifies:

- the model endpoint or provider
- the orchestration layer
- the system prompt and trusted instruction sources
- memory stores and retrieval paths
- tools and external systems the agent can invoke
- approval points and human operators
- identities used by the agent and each supporting component
- logging, monitoring, and emergency shutdown paths

## Identity and access baseline

### Microsoft Entra ID and workload identity

The organisation MUST use Microsoft Entra ID as the primary identity plane for users, administrators, service principals, and managed identities wherever Azure-native integration is available.

Minimum requirements:

- MFA MUST be enforced for all human users.
- Phishing-resistant MFA MUST be used for privileged roles unless an approved exception exists with compensating controls.
- Conditional Access MUST be configured for user sign-ins covering administrators, high-risk sign-ins, unmanaged devices, guest users, and sensitive applications.
- Conditional Access applies to user sign-ins and does not enforce service principal or managed identity authentication flows. Workload identity access MUST therefore be constrained separately.
- Managed identities MUST be used for Azure-hosted agent components wherever supported.
- Long-lived shared secrets for agent tools, orchestration services, or external APIs MUST be avoided wherever federation, certificates, or managed identities can be used instead.
- Break-glass access for the platform MUST be tightly controlled, monitored, and tested at a defined cadence.

### Agent identities as non-human identities

Each agent or agent execution context MUST have an identity model that is:

- distinct from human operator identities
- scoped to a defined set of tools and resources
- time-bounded where possible
- separated by environment and trust boundary

The organisation MUST NOT:

- reuse one broad service principal across unrelated agents
- share production agent credentials with development or test agents
- allow one agent to inherit a human administrator's standing privileges

### App roles, RBAC, and tool authorization

These control layers MUST remain separate:

- Entra directory roles govern tenant and identity administration
- Azure RBAC governs Azure management-plane access
- application roles and scopes govern business actions inside applications and APIs
- tool authorization policies govern which tools an agent may invoke and with what arguments

Required design rules:

- Azure RBAC MUST NOT be used as the sole authorization model for agent actions inside business systems.
- Agent tool calls MUST be validated by application-level policy, not merely by possession of network connectivity or an Azure role.
- Azure Contributor, Owner, or similar management-plane access MUST NOT automatically grant application-level or agent-level privileges.
- High-risk tools MUST require separate approval and narrower authorization than low-risk read-only tools.

## Azure infrastructure baseline for agentic AI

### Governance and landing zones

All agentic AI workloads MUST be deployed within a governed Azure landing-zone model with:

- separate production and non-production subscriptions or equivalent isolation
- mandatory tagging for owner, environment, data classification, agent type, and business criticality
- Azure Policy for approved regions, network exposure, diagnostics, identity settings, and encryption
- approved service patterns for model access, memory stores, tool APIs, and operator consoles

### Network architecture

Minimum requirements:

- Internet-facing agentic applications MUST publish through approved edge services such as Azure Front Door or Application Gateway WAF.
- Direct public exposure of agent control planes, memory stores, vector stores, admin consoles, databases, queues, and internal tool APIs is prohibited unless explicitly approved.
- Private endpoints SHOULD be used for storage, databases, AI services, and internal APIs where supported.
- Administrative access MUST be brokered through approved access paths such as Bastion, JIT, VPN, or equivalent controls.
- East-west communication between agent components MUST be segmented and governed through network controls, service authentication, and explicit allow rules.

### Compute and runtime isolation

Agent runtimes MUST be deployed with containment controls appropriate to the risk of tool invocation and code execution.

The organisation SHOULD:

- isolate high-risk agent workloads from general application hosting
- use sandboxed or strongly constrained execution environments for code generation or code execution features
- separate approval services from execution services
- separate memory services from action-execution services where practical

### Secrets and cryptography

Required controls:

- Secrets, keys, and certificates MUST be stored in Azure Key Vault or an approved equivalent.
- Agent runtimes MUST NOT embed secrets in prompts, code, local files, or images.
- Encryption at rest MUST be enabled for supported services.
- TLS 1.3 SHOULD be preferred. TLS 1.2 MAY be used where required for compatibility and approved by security review. TLS versions below 1.2 MUST be disabled.

### Logging, monitoring, and detection

All production agentic systems MUST log enough information to reconstruct:

- user requests and trusted operator inputs
- model requests and responses where lawful and appropriate
- prompt assembly and retrieval sources where needed for investigation
- tool selections, parameters, and execution outcomes
- memory reads and writes
- goal or plan changes
- human approvals, overrides, and rejections
- identity events, role changes, and privileged operations

Microsoft Entra sign-in, audit, provisioning, and consent-related logs MUST be centrally collected and retained in line with organisational requirements.

## Application baseline for agentic AI

### Prompt and instruction boundary design

Applications MUST clearly separate:

- trusted system instructions
- developer-controlled orchestration rules
- retrieved external content
- user input
- tool outputs
- memory content

Untrusted content MUST NOT be treated as authoritative instruction without explicit validation and policy.

### Tooling model

Every tool exposed to an agent MUST define:

- purpose
- owner
- risk category
- allowed caller identities
- allowed parameters and schema
- destructive or externally visible side effects
- approval requirements
- rate limits and containment actions

### Human approval model

Human approvals MUST be treated as a first-class security control. Approval interfaces MUST:

- show the actual action proposed
- show target systems, scope, and affected resources
- avoid persuasive or misleading summaries without evidence
- require step-up approval for high-impact actions where warranted
- maintain immutable audit logs

### Memory and retrieval model

Memory and retrieval components MUST define:

- who can write memory
- what content types can be written
- how provenance is tracked
- how stale or poisoned memory is corrected or expired
- how tenant, user, and session boundaries are enforced

## OWASP Top 10 for Agentic Applications mapping

This baseline incorporates the OWASP Top 10 for Agentic Applications (2026), using the `ASI01` to `ASI10` identifiers and OWASP public terminology where available, with the control summaries written for Azure implementation use.

### ASI01: Agent Behavior Hijacking

Risk summary:

- Untrusted content changes the agent's goal, plan, or execution path.

Azure, infrastructure, and application requirements:

- Trusted instructions MUST be stored separately from untrusted content.
- Goal changes for high-impact workflows MUST trigger approval or containment.
- Retrieval sources from Azure AI Search, storage, or external content feeds MUST be treated as untrusted unless validated.
- Internet-facing agents MUST implement prompt abuse detection, content filtering, and plan-change monitoring.

### ASI02: Tool Misuse

Risk summary:

- Agents misuse legitimate tools in destructive or unintended ways.

Azure, infrastructure, and application requirements:

- Tool execution MUST be constrained by explicit allowlists and parameter validation.
- Destructive actions MUST require separate authorization and, where appropriate, human approval.
- Tool APIs SHOULD be fronted by an application policy layer or API gateway instead of exposing raw direct access.
- Write-capable tools SHOULD support dry-run or draft-only modes.

### ASI03: Identity & Privilege Abuse

Risk summary:

- Agents inherit, escalate, or misuse credentials and privileges.

Azure, infrastructure, and application requirements:

- Agents MUST use task-scoped managed identities or short-lived federated or certificate-based credentials.
- Privileged Azure RBAC and application permissions MUST be explicitly separated from operator access.
- Per-action authorization checks MUST be performed by downstream applications and APIs.
- Standing broad access for agent identities is prohibited unless risk accepted and documented.

### ASI04: Agentic Supply Chain Vulnerabilities

Risk summary:

- Compromised dependencies, tools, prompts, registries, or MCP-compatible components introduce malicious behavior.

Azure, infrastructure, and application requirements:

- Agent dependencies, prompts, models, connectors, and tool endpoints MUST be inventoried.
- Approved versions and sources SHOULD be pinned and integrity-checked where supported.
- Third-party MCP servers, plugins, or remote tools MUST undergo security review before production use.
- The organisation SHOULD maintain an AI-focused bill of materials for significant agentic systems.

### ASI05: Unexpected Code Execution

Risk summary:

- Agent-generated or agent-invoked code executes unsafely.

Azure, infrastructure, and application requirements:

- Model output MUST NOT be executed directly without validation and containment.
- Code execution features MUST run in isolated environments with limited privileges, egress controls, and monitoring.
- High-risk execution environments SHOULD be separated from production control planes and sensitive data stores.
- Executable artifacts generated by agents MUST be subject to policy and security validation before promotion.

### ASI06: Memory & Context Poisoning

Risk summary:

- Malicious or incorrect content is written into memory or context and influences later behavior.

Azure, infrastructure, and application requirements:

- Memory MUST be segmented by tenant, user, session, and trust boundary as appropriate.
- Memory writes MUST be validated for malicious patterns, instruction smuggling, and prohibited data.
- Memory entries SHOULD have provenance, expiration, and rollback capability.
- High-impact decisions SHOULD NOT rely solely on long-lived agent memory without corroboration.

### ASI07: Insecure Inter-Agent Communication

Risk summary:

- Messages between agents or orchestration components are spoofed, tampered with, or misrouted.

Azure, infrastructure, and application requirements:

- Inter-agent communication MUST use authenticated service identities and encrypted transport.
- Message schemas MUST be validated and trust boundaries explicitly defined.
- Agent discovery and routing MUST be controlled, not implicit.
- Multi-agent systems SHOULD use zero-trust assumptions between agents even inside one environment.

### ASI08: Cascading Failures

Risk summary:

- One faulty action or agent failure propagates through a wider workflow.

Azure, infrastructure, and application requirements:

- Agent workflows MUST enforce quotas, timeouts, retry limits, and fan-out controls.
- Circuit breakers and kill switches MUST exist for agents, tools, and workflow stages.
- Containment MUST be possible without shutting down the entire environment where feasible.
- Monitoring MUST detect abnormal chaining, runaway invocation, or repeated failure conditions.

### ASI09: Human-Agent Trust Exploitation

Risk summary:

- Humans are manipulated into approving harmful actions or trusting unsafe outputs.

Azure, infrastructure, and application requirements:

- Approval UIs MUST show evidence, scope, and impact clearly.
- High-risk actions SHOULD require step-up authentication or dual control where warranted.
- The system MUST NOT hide material uncertainty, scope expansion, or destructive side effects behind friendly summaries.
- Audit logs MUST preserve the proposed action, approval identity, and resulting execution details.

### ASI10: Rogue Agents

Risk summary:

- Malicious, compromised, or drifting agents deviate from intended behavior.

Azure, infrastructure, and application requirements:

- Production agents MUST have behavior baselines, anomaly detection, and revocation procedures.
- The organisation MUST be able to disable tools, revoke identities, quarantine workloads, and stop workflows quickly.
- Prompt, memory, tool, and model changes SHOULD be traceable and attributable.
- Autonomous agents with high-impact privileges MUST have stronger oversight than read-only or advisory agents.

## Control expectations by deployment type

| Area | Internal-only agentic system | Internet-facing agentic system |
|------|-------------------------------|--------------------------------|
| Entra authentication | Required | Required |
| Managed identity / workload identity control | Required | Required |
| Tool allowlists | Required | Required |
| Human approval for destructive actions | Required | Required |
| Private endpoints for internal services | Strongly preferred | Strongly preferred for back-end services |
| WAF / edge protection | As needed | Required |
| Abuse rate limiting | Required by risk | Required |
| Model and tool telemetry | Required | Required |
| Memory provenance and segmentation | Required | Required |
| Circuit breakers / kill switches | Required | Required |
| External content trust controls | Required | Required with heightened scrutiny |

## Azure implementation guardrails

The organisation SHOULD:

- use Azure API Management, gateway logic, or equivalent policy enforcement in front of tool APIs
- use Azure Key Vault for credentials and signing materials
- use managed identities for agent runtimes on App Service, Functions, AKS, Container Apps, and VMs where supported
- use private networking for storage, vector stores, queues, and databases where feasible
- isolate code execution workloads from operator consoles and memory services
- use Defender for Cloud, Azure Monitor, and Microsoft Sentinel or equivalent capabilities for visibility and detection
- define Azure Policy guardrails for diagnostic settings, public exposure, approved regions, and identity settings

## Minimum evidence the organisation should retain

The organisation SHOULD be able to produce:

- architecture diagrams for the agentic system and trust boundaries
- inventory of models, prompts, tools, memory stores, and third-party components
- identity design for users, operators, and agent workloads
- tool authorization model and approval rules
- logging and monitoring design
- memory governance and provenance design
- incident response and containment procedures
- test evidence for high-risk scenarios such as tool misuse, memory poisoning, and cascading failure
- exception records with approval, expiry, and remediation plan

## Non-compliance and exceptions

Any exception to this baseline MUST:

- be documented
- state the control not met
- include risk rationale and compensating controls
- identify an accountable owner
- have an approval record
- have an expiry or review date

Expired exceptions MUST be remediated or formally re-approved. Re-approval history SHOULD be retained in a central exception register.

## Summary

In practical terms, this baseline requires the organisation to:

- secure agent identities as non-human identities with least privilege
- treat tools, memory, retrieval, and approvals as security boundaries
- isolate Azure infrastructure so that one compromised agent does not become a platform-wide incident
- design applications so that agent autonomy is bounded, observable, and reversible
- map Azure controls directly to the OWASP Top 10 for Agentic Applications instead of treating agent risk as only a prompt-security issue

Meeting this baseline does not guarantee security on its own, but it creates a strong Azure-focused foundation for reducing the most important agentic AI risks identified by OWASP.

## References

- OWASP GenAI Security Project: https://genai.owasp.org/
- OWASP Top 10 for Agentic Applications for 2026: https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/
- OWASP announcement, Top 10 for Agentic Applications: https://genai.owasp.org/2025/12/09/owasp-top-10-for-agentic-applications-the-benchmark-for-agentic-security-in-the-age-of-autonomous-ai/
- OWASP Agentic AI Threats and Mitigations: https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/

Note: the primary OWASP resource page is concise. This document uses OWASP public terminology where available and Azure-focused control summaries to make the risks actionable for implementation.
