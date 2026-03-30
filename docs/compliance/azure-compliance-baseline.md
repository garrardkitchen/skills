# Azure Compliance Baseline for ISO 27001, GDPR, UK GDPR, and EU AI Act

## Status

Draft baseline standard

## Contents

- [Purpose](#purpose)
- [Governance and adoption](#governance-and-adoption)
- [Scope](#scope)
- [Key principles](#key-principles)
- [Applicability model](#applicability-model)
  - [Internet-facing services](#internet-facing-services)
  - [Internal-only services](#internal-only-services)
- [Identity and access baseline](#identity-and-access-baseline)
  - [Microsoft Entra ID and Microsoft identity platform](#microsoft-entra-id-and-microsoft-identity-platform)
  - [App roles, groups, and Azure RBAC](#app-roles-groups-and-azure-rbac)
  - [Preventing role confusion and authorization bypass](#preventing-role-confusion-and-authorization-bypass)
  - [Joiner, mover, leaver requirements](#joiner-mover-leaver-requirements)
- [Azure resource and infrastructure baseline](#azure-resource-and-infrastructure-baseline)
  - [Governance and landing zones](#governance-and-landing-zones)
  - [Network architecture](#network-architecture)
  - [Platform hardening](#platform-hardening)
  - [Secrets and cryptography](#secrets-and-cryptography)
  - [Logging, monitoring, and detection](#logging-monitoring-and-detection)
- [Application baseline](#application-baseline)
  - [Secure application design](#secure-application-design)
  - [Internet-facing application requirements](#internet-facing-application-requirements)
  - [Internal application requirements](#internal-application-requirements)
  - [DevSecOps and delivery](#devsecops-and-delivery)
- [Privacy baseline for GDPR and UK GDPR](#privacy-baseline-for-gdpr-and-uk-gdpr)
  - [Data protection governance](#data-protection-governance)
  - [Privacy by design requirements](#privacy-by-design-requirements)
  - [International transfer and residency considerations](#international-transfer-and-residency-considerations)
  - [DPIA triggers](#dpia-triggers)
- [ISO 27001-specific requirements](#iso-27001-specific-requirements)
- [GDPR-specific requirements](#gdpr-specific-requirements)
- [UK GDPR-specific requirements](#uk-gdpr-specific-requirements)
- [EU AI Act baseline](#eu-ai-act-baseline)
  - [Applicability](#applicability)
  - [AI governance requirements](#ai-governance-requirements)
  - [Prohibited AI practices](#prohibited-ai-practices)
  - [Technical and operational controls](#technical-and-operational-controls)
  - [High-risk and sensitive use cases](#high-risk-and-sensitive-use-cases)
  - [Transparency requirements](#transparency-requirements)
- [Control expectations by service type](#control-expectations-by-service-type)
- [Minimum evidence the organisation should retain](#minimum-evidence-the-organisation-should-retain)
- [Recommended implementation guardrails in Azure](#recommended-implementation-guardrails-in-azure)
- [Non-compliance and exceptions](#non-compliance-and-exceptions)
- [Document review triggers](#document-review-triggers)
- [Summary](#summary)
- [References](#references)

## Purpose

This document defines the minimum control baseline for Azure-hosted resources, infrastructure, identity services, and applications used:

- via the public internet
- internally within an organisation
- by employees, contractors, partners, and service-to-service workloads

It is written as a reusable Azure baseline aligned to:

- ISO/IEC 27001
- GDPR
- UK GDPR
- EU AI Act

This document is a technical and operational baseline intended to support compliance design, implementation, and assurance. It should be reviewed by security, privacy, legal, and architecture stakeholders before being adopted as a binding organisational standard.

## Governance and adoption

This document is a reference baseline until formally adopted by the organisation's governance process.

Before this document is used as a binding standard, the organisation SHOULD define:

- the approving authority for the baseline
- the control owners accountable for implementation
- the exception and waiver process
- the review cadence
- the relationship between this baseline and legal, privacy, procurement, and records-management processes

Once adopted, the organisation MAY use the mandatory language in this document as internal policy language. Until then, it SHOULD be treated as a template or reference standard.

## Scope

This baseline applies to:

- Microsoft Entra ID and the Microsoft identity platform
- Azure subscriptions, management groups, policies, and landing zones
- Azure networking, perimeter, and private connectivity
- Azure compute, platform, data, storage, integration, and AI services
- internet-facing and internal-only applications
- APIs, web apps, background services, batch jobs, and automation
- administrators, developers, operators, and third-party support personnel

## Key principles

- Compliance requirements apply to the whole service, not just the application code.
- Internet-facing systems require stronger default controls than internal-only systems.
- Privacy, security, and AI governance must be designed into the platform, not added later.
- Microsoft Entra roles, Azure RBAC, and application roles serve different purposes and must not be conflated.
- The organisation must be able to produce evidence that controls are defined, implemented, monitored, and reviewed.

## Applicability model

### Internet-facing services

Services reachable from the public internet, directly or through an internet-exposed endpoint, MUST implement the full baseline with heightened controls for:

- edge protection
- authentication strength
- telemetry and alerting
- vulnerability and patch management
- secrets handling
- DDoS, bot, and abuse resistance

Service classification as internet-facing MUST be documented, approved by the service owner, and reviewed whenever exposure changes.

### Internal-only services

Services intended only for internal use MUST still implement the baseline, but may rely more heavily on:

- private networking
- trusted device and user conditions
- segmented internal access paths
- restricted exposure through VPN, ExpressRoute, Bastion, or equivalent controls

Service classification as internal-only MUST be documented, approved, and reviewed periodically. Any change in exposure model MUST trigger security and compliance reassessment before go-live.

Internal-only status MUST NOT be used as justification to omit:

- strong identity controls
- least privilege
- logging
- encryption
- patching
- backup and recovery
- data protection controls

## Identity and access baseline

### Microsoft Entra ID and Microsoft identity platform

The organisation MUST use Microsoft Entra ID as the authoritative identity plane for workforce identities, workload identities, and application registrations where Azure-native integration is available.

Minimum requirements:

- MFA MUST be enforced for all human users. Phishing-resistant MFA MUST be used for privileged roles unless a documented exception with compensating controls has been approved.
- Conditional Access MUST be configured for user sign-ins covering administrators, high-risk sign-ins, unmanaged devices, guest users, and sensitive applications. Where relevant, risky locations and device compliance conditions SHOULD also be enforced.
- Conditional Access applies to user sign-ins and does not enforce service principal or managed identity authentication flows. Workload identity access MUST therefore be constrained separately through managed identities, certificate or federation-based authentication, RBAC, and network controls as applicable.
- Legacy authentication MUST be blocked unless an approved exception exists.
- Named accounts MUST be used for administrative access; shared admin identities are prohibited.
- Break-glass accounts MUST be tightly controlled, monitored, excluded from Conditional Access only where required for emergency access design, and tested at a defined cadence.
- Privileged Identity Management SHOULD be used for privileged Entra and Azure roles.
- Access Reviews MUST be used for privileged roles, external users, and high-sensitivity applications.
- External guest access MUST be governed through explicit approval, least-privilege assignment, expiry where appropriate, and periodic review.
- Workload identities MUST use managed identities for Azure-hosted workloads wherever the service and integration pattern support them.
- Certificates or federated credentials MUST be preferred over long-lived shared secrets when managed identities cannot be used. Federated credentials SHOULD be used for external CI/CD platforms that support workload identity federation, while certificate-based identities SHOULD be used for non-Azure hosted workloads or platforms that cannot use federation.

### App roles, groups, and Azure RBAC

These control types MUST be separated by purpose:

- Microsoft Entra directory roles control tenant-level identity administration.
- Azure RBAC controls access to Azure management plane resources.
- Application roles control what a signed-in user or calling service can do inside an application.
- Security groups may be used to assign application roles or simplify access management, but they do not replace application authorization design.

Required design rules:

- Azure RBAC MUST NOT be used as the sole authorization model inside business applications.
- Internet-facing and internal applications MUST define explicit application roles for privileged or sensitive business functions.
- Application roles MUST be assigned by approved administrators or governed group membership.
- Service-to-service APIs MUST validate issuer, audience, tenant, and role or scope claims.
- Multi-tenant applications MUST be explicitly approved and threat-assessed before use, including review of data isolation, shared infrastructure risk, and blast radius.
- Consent to Microsoft identity platform permissions MUST be governed and auditable.
- High-privilege Graph API permissions MUST require explicit security review.

### Preventing role confusion and authorization bypass

The organisation MUST maintain clear separation between management-plane access and application-plane access:

- Azure RBAC grants to developers, operators, and automation MUST NOT be used to bypass application authorization controls.
- Administrative identities with Contributor, Owner, or equivalent access MUST NOT receive unnecessary direct access to application data stores.
- Application authorization decisions MUST be based on application roles, scopes, or application-specific policy, not Azure RBAC role names.
- Service-to-service APIs MUST validate application roles or scopes independently of any Azure RBAC permissions granted to the calling workload identity.
- Microsoft Graph delegated permissions, Microsoft Graph application permissions, Entra directory roles, Azure RBAC roles, and application roles MUST be treated as separate control layers with separate approval paths.
- Azure Contributor, Owner, or similar management-plane access MUST NOT automatically grant application-level privileges. Application roles MUST be assigned separately based on business need.

Practical interpretation:

- Entra directory roles control directory administration.
- Azure RBAC controls Azure resource administration.
- Microsoft Graph permissions control what an app or user can do through Graph.
- Application roles and scopes control what a user or service can do inside the application itself.

### Joiner, mover, leaver requirements

The organisation MUST ensure:

- timely provisioning and deprovisioning
- removal of privileged access on role change
- periodic recertification of access
- termination of external guest access when no longer required

These controls support ISO 27001 access control requirements and GDPR/UK GDPR integrity and confidentiality obligations.

## Azure resource and infrastructure baseline

### Governance and landing zones

All Azure resources MUST be deployed within a governed landing-zone model with:

- defined management group hierarchy
- approved subscription structure
- mandatory tagging for owner, data classification, environment, criticality, and business service
- Azure Policy for allowed locations, SKUs, diagnostics, encryption, networking, and identity settings
- resource locks where required for critical services
- separate production and non-production environments

### Network architecture

Minimum network requirements:

- Internet-facing workloads MUST terminate through approved edge services such as Azure Front Door, Application Gateway with WAF, or an equivalent managed control.
- Direct public exposure of data stores, management interfaces, and administrative endpoints is prohibited unless explicitly approved.
- Internal-only applications SHOULD use private endpoints, private DNS, and segmented virtual networks.
- NSGs, firewalls, and route controls MUST be used to restrict east-west and north-south traffic.
- Administrative access MUST be brokered through controlled paths such as Bastion, JIT access, VPN, or equivalent.
- DDoS protection MUST be implemented for internet-facing critical services unless a documented exception with approved risk acceptance exists.
- TLS 1.3 SHOULD be preferred. TLS 1.2 MAY be used where required for compatibility and approved by security review. TLS versions below 1.2 MUST be disabled.

For the purposes of this baseline, public management endpoints include direct internet-reachable administrative interfaces such as SSH, RDP, Kubernetes control-plane access, database administration ports, and application or platform management interfaces not brokered through approved access paths.

### Platform hardening

The organisation MUST:

- maintain supported runtimes and operating systems
- patch critical vulnerabilities within defined service-level targets
- enable endpoint and workload protection where supported
- disable unused ports, protocols, features, and default accounts
- restrict outbound connectivity where feasible
- maintain secure build images and hardened base configurations

### Secrets and cryptography

Required controls:

- Secrets, keys, and certificates MUST be stored in approved secret-management services such as Azure Key Vault.
- Secrets MUST NOT be stored in code, pipeline variables in plain text, or local configuration files without approved protection.
- Encryption at rest MUST be enabled for supported services.
- Key rotation and certificate renewal processes MUST be defined and monitored.
- Customer-managed keys SHOULD be used when required by risk assessment, contractual obligation, or data sensitivity. Organisations using customer-managed keys MUST confirm service support, operational ownership, backup, and recovery implications before adoption.

### Logging, monitoring, and detection

All production services MUST emit auditable logs for:

- authentication and authorization events
- privileged actions
- configuration changes
- resource creation and deletion
- network security events
- application security events
- AI-related high-risk actions where applicable

Required capabilities:

- central log collection
- time synchronization
- alerting for high-severity events
- retention aligned to legal, security, and business requirements
- tamper-resistant storage or access restrictions for sensitive logs

Microsoft Entra ID sign-in, audit, provisioning, and consent-related logs MUST be centrally collected and retained in line with organisational requirements.

## Application baseline

### Secure application design

Applications MUST:

- authenticate through approved identity providers, normally Microsoft Entra ID
- implement server-side authorization checks for every sensitive operation
- use application roles or scoped permissions for business authorization
- validate input, output, file upload, and deserialization boundaries
- protect session tokens, cookies, and API credentials
- encrypt sensitive data in transit and at rest
- segregate duties for administration, support, and business users

### Internet-facing application requirements

Internet-facing applications MUST additionally:

- use approved WAF and edge protection
- protect against OWASP Top 10 classes of weakness
- implement rate limiting, abuse controls, and bot protection where relevant
- avoid direct internet exposure of backend data services
- expose health and diagnostics endpoints only in a controlled manner
- perform external attack-surface review and vulnerability scanning

### Internal application requirements

Internal applications MUST:

- still require authenticated access
- avoid assuming that network location alone is sufficient trust
- restrict high-risk functions to approved roles
- log privileged and risk-significant business actions
- use private access paths where practical

The logging depth, vulnerability-scanning cadence, and hardening assurance for internal-only services MAY be risk-adjusted, but the organisation MUST document the rationale and minimum scanning frequency for each service class. This risk adjustment applies to cadence and depth, not to the presence of foundational controls such as authentication, encryption, patching, backup, and least privilege.

### DevSecOps and delivery

The delivery process MUST include:

- code review
- dependency and vulnerability scanning
- secrets scanning
- environment segregation
- controlled deployment approvals for production
- rollback and recovery procedures
- evidence of who changed what and when

Detected secrets MUST be treated as compromised, rotated, and remediated in source history or configuration where required.

## Privacy baseline for GDPR and UK GDPR

### Data protection governance

Where personal data is processed, the organisation MUST define:

- controller and processor roles
- lawful basis for processing
- data categories and data subjects
- purpose limitation
- retention periods
- deletion and rectification processes
- records of processing where required

### Privacy by design requirements

Azure resources and applications handling personal data MUST:

- minimize personal data collected, stored, and exposed
- separate production personal data from test and development environments
- pseudonymize or anonymize data where possible
- restrict access based on need to know
- log access to sensitive personal data where proportionate and lawful
- support data subject rights processes, including access, deletion, rectification, restriction, and portability where applicable

Where support or operational access by Microsoft or other providers may expose personal data, the organisation MUST assess processor risk, verify contractual data processing terms, and use additional controls such as Customer Lockbox, segregation, or pseudonymization where appropriate.

### International transfer and residency considerations

The organisation MUST assess:

- where data is stored and processed
- whether support or operational access crosses borders
- whether Microsoft support, administrative access, identity telemetry, logging backends, or security operations processing introduce cross-border access
- whether backups, replication, or vendor-operated service functions introduce cross-border processing outside the primary deployment region
- whether international transfer mechanisms are required
- whether customer or regulatory residency commitments apply

For UK GDPR and GDPR, transfer mechanisms, contracts, and vendor terms MUST align to the applicable jurisdictional requirements.

Where cross-border transfers occur, the organisation MUST assess whether transfer impact assessments and supplementary technical, contractual, or organisational measures are required.

### DPIA triggers

A Data Protection Impact Assessment MUST be completed before go-live where processing is likely to result in high risk, including where it involves:

- large-scale personal data
- special category data
- systematic monitoring
- vulnerable individuals
- new technology with elevated privacy risk
- profiling or automated decision-making with legal or similarly significant effects
- AI or ML features processing personal data for profiling, predictions, recommendations, automated decision-making, or systematic monitoring

The organisation SHOULD define who performs the DPIA review, where the decision is recorded, and what approval is required before deployment proceeds.

## ISO 27001-specific requirements

To support ISO 27001 alignment, the organisation MUST maintain an ISMS-supported control environment around Azure and applications, including:

- documented scope for in-scope services and supporting assets
- risk assessment and treatment records
- statement of applicability
- asset inventory
- control ownership
- supplier and third-party risk management
- incident management procedures
- backup, recovery, and continuity testing
- change management and access management procedures
- security awareness and role-specific training

Azure implementation implications:

- every production service MUST have an identified owner and classification
- every critical service MUST have backup and recovery arrangements that are tested at a defined cadence and produce recovery evidence
- privileged access MUST be reviewable and time-bound where possible
- security events MUST feed into an incident management process
- exceptions MUST be documented, approved, time-limited, and reviewed

## GDPR-specific requirements

To support GDPR alignment, services processing EU personal data MUST implement controls demonstrating:

- lawfulness, fairness, and transparency
- purpose limitation
- data minimization
- accuracy
- storage limitation
- integrity and confidentiality
- accountability

Azure and application implications:

- data flows involving personal data MUST be documented
- privacy notices and in-product transparency MUST match actual processing
- processors and subprocessors MUST be governed through binding contracts, including data processing terms that meet GDPR Article 28 and equivalent UK GDPR requirements
- breach detection, triage, and notification processes MUST exist and support notification to the relevant supervisory authority within required legal timeframes where applicable
- automated decision-making and profiling MUST be assessed before use

## UK GDPR-specific requirements

Many technical controls for UK GDPR will overlap with GDPR, but the organisation MUST separately assess UK-specific legal and governance obligations where UK personal data is involved, including:

- UK-specific transfer mechanisms where applicable
- UK International Data Transfer Agreement or UK Addendum requirements where applicable
- UK supervisory authority expectations
- UK Data Protection Act 2018 obligations relevant to the service context
- contractual wording and vendor terms covering UK transfers
- whether the service needs UK-specific privacy governance, notices, or records

Operationally, UK GDPR MUST NOT be assumed to be satisfied solely because GDPR controls exist; jurisdictional review is still required.

## EU AI Act baseline

### Applicability

The EU AI Act applies according to the role performed by the organisation, such as provider, deployer, importer, distributor, or product manufacturer, and according to the risk classification of the AI system.

This baseline applies when Azure AI services, Azure OpenAI, custom ML models, model-assisted decisioning, or embedded AI features are used in ways that may place the capability within the scope of the EU AI Act or materially affect people, regulated decisions, or rights-sensitive outcomes.

### AI governance requirements

Before deployment, the organisation MUST determine:

- whether the capability is an AI system in scope
- whether the use case is prohibited, high-risk, limited-risk, or lower-risk
- whether the organisation acts as provider, deployer, or both
- whether personal data is involved
- whether the output materially influences decisions about individuals

Simple deterministic rules, conventional analytics, non-adaptive dashboards, and basic automation SHOULD NOT be treated as in-scope AI use cases solely because they are software-driven; a threshold assessment referencing the EU AI Act definition and relevant prohibited or high-risk categories SHOULD be completed instead.

### Prohibited AI practices

The organisation MUST confirm that the proposed AI use case does not fall into a prohibited practice category under the EU AI Act before development or deployment proceeds. Any use case that appears to involve prohibited manipulation, exploitative use against vulnerable persons, unlawful social scoring, or prohibited biometric use cases MUST be escalated for legal review and blocked unless lawfully justified.

### Technical and operational controls

Where AI is used, the organisation MUST implement:

- documented use case and intended purpose
- named business and technical owners
- risk assessment before production use
- data provenance and dataset governance appropriate to the use case
- human oversight arrangements for material decisions
- logging sufficient to investigate output, misuse, incidents, and drift
- security testing for prompt injection, data leakage, abuse, and model misuse where relevant
- output handling rules for downstream systems and operators
- incident escalation and model rollback or disablement procedures

### High-risk and sensitive use cases

Where the AI use case may fall into a high-risk category, the service MUST NOT go live until legal, compliance, security, privacy, and business review confirms that required obligations are met. Depending on the role and use case, those obligations may include:

- risk management system
- technical documentation
- data governance controls
- logging
- human oversight
- accuracy, robustness, and cybersecurity measures
- transparency and user instructions
- the applicable EU AI Act conformity assessment procedure before placing the system on the market or putting it into service, with notified body involvement and CE marking where legally required

### Transparency requirements

Applications using AI SHOULD disclose AI involvement to users where required by law, policy, or risk assessment, especially when:

- users may believe they are interacting only with a human
- generated content may require labeling
- outputs materially influence decisions or actions

## Control expectations by service type

| Area | Internal-only service | Internet-facing service |
|------|------------------------|-------------------------|
| Entra authentication | Required | Required |
| MFA for users | Required | Required |
| Conditional Access | Required | Required with stronger conditions |
| App roles for privileged functions | Required | Required |
| Azure RBAC least privilege | Required | Required |
| Private endpoints | Preferred | Strongly preferred for back-end services |
| Public management endpoints | Prohibited unless approved | Prohibited unless approved |
| WAF / edge protection | As needed | Required |
| DDoS protection | As needed based on risk | Required for critical exposure |
| Central logging | Required | Required |
| Vulnerability scanning | Required at risk-appropriate cadence | Required with external attack-surface focus and faster cadence |
| DPIA review when personal data risk is high | Required where triggered | Required where triggered |
| AI risk classification when AI is used | Required | Required |

## Minimum evidence the organisation should retain

The organisation SHOULD be able to produce:

- architecture diagrams and data-flow diagrams
- asset inventory and service ownership records
- access model, app role definitions, and RBAC assignments
- Conditional Access and privileged access design
- vulnerability, patching, and hardening evidence
- logging and alerting configuration
- backup and restore test evidence
- DPIAs, transfer assessments, and privacy records where personal data is processed
- AI risk assessments, approval records, and monitoring evidence where AI is used
- exception records with approval, expiry, and remediation plan

## Recommended implementation guardrails in Azure

- Use management groups, Azure Policy, and policy exemptions with approval workflow.
- Use separate subscriptions for production, non-production, and shared services.
- Use Microsoft Entra groups for assignment hygiene, but keep application authorization in application roles.
- Use managed identities for Azure-hosted workloads.
- Use workload identity federation for external CI/CD platforms where supported, and avoid long-lived client secrets.
- Use Key Vault for secrets, keys, and certificates.
- Use Defender for Cloud and relevant workload protection plans based on risk.
- Use Front Door or Application Gateway WAF for internet-facing applications.
- Use private endpoints for storage, databases, and PaaS services where feasible.
- Use Log Analytics, Microsoft Sentinel, or equivalent SIEM/SOAR capability for central visibility where required by risk.

## Non-compliance and exceptions

Any exception to this baseline MUST:

- be documented
- state the specific control not met
- include risk rationale and compensating controls
- identify an accountable owner
- have an approval record
- have an expiry or review date

Expired exceptions MUST be remediated or formally re-approved. Re-approval history SHOULD be retained in a central exception register.

Unapproved deviation from this baseline SHOULD be treated as a compliance and security issue.

## Document review triggers

This baseline SHOULD be reviewed when:

- a new Azure service pattern is introduced
- a new internet-facing system is proposed
- a high-risk internal system is designed
- personal data processing materially changes
- AI capability is introduced or materially changed
- a new legal, contractual, or regulatory requirement applies

## Summary

In practical terms, this baseline requires the organisation to:

- govern Azure through policy, ownership, and evidence
- secure identity through Entra ID, strong authentication, app roles, and least-privilege RBAC
- protect internet-facing and internal systems with different exposure assumptions but the same core security discipline
- build privacy controls into architecture and operations when personal data is processed
- treat AI use as a governed capability requiring explicit risk classification, oversight, and monitoring

Meeting this baseline does not guarantee certification or legal compliance on its own, but it creates a strong technical and operational foundation for ISO 27001, GDPR, UK GDPR, and EU AI Act readiness.

## References

The following UK public-sector and legislation sources inform this baseline:

- ISO 27001 readiness context:
  - NCSC Cyber Assessment Framework (CAF): https://www.ncsc.gov.uk/collection/cyber-assessment-framework
  - NCSC 10 Steps to Cyber Security: https://www.ncsc.gov.uk/collection/10-steps-to-cyber-security

- GDPR and UK GDPR legislation:
  - UK GDPR / assimilated Regulation (EU) 2016/679 contents on legislation.gov.uk: https://www.legislation.gov.uk/eur/2016/679/contents
  - UK GDPR Article 5, Principles relating to processing of personal data: https://www.legislation.gov.uk/eur/2016/679/article/5
  - UK GDPR Article 32, Security of processing: https://www.legislation.gov.uk/eur/2016/679/article/32
  - Data Protection Act 2018: https://www.legislation.gov.uk/ukpga/2018/12/contents

- EU AI Act readiness context:
  - UK Government, A pro-innovation approach to AI regulation: https://www.gov.uk/government/publications/ai-regulation-a-pro-innovation-approach

Note: the EU AI Act itself is EU legislation rather than a UK government publication. The UK Government AI regulation paper above is included here as UK public-sector readiness context, not as the authoritative legal text of the EU AI Act.
