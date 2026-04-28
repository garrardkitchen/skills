# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Added `agentic-security-assessment/SKILL.md`, a reusable skill that assesses Terraform and application code against `ASI01` through `ASI10` in the Azure Agentic AI Security Baseline, generates a markdown assessment report, creates a styled Mermaid diagram, and documents gaps with technology-specific implementation links.
- Added `azure-compliance-assessment/SKILL.md`, a reusable skill that assesses or proves alignment against `docs/compliance/azure-compliance-baseline.md`, asks the user to choose assessment or proof mode, generates emoji-based markdown compliance tables, and produces a styled Mermaid diagram showing where controls are addressed across infrastructure and application layers.
- Added `dotnet-agentic-security/`, a modular skill for secure `.NET` and `ASP.NET Core` development that covers OWASP agentic and web risks, HTTP APIs, Microsoft Agent Framework workflows, `Azure.Identity`, architecture patterns, data access, testing, observability, and script-style app guidance through progressively loaded reference markdown files.
- Added `docs/compliance/azure-compliance-baseline.md`, a draft policy-style Azure compliance baseline covering ISO 27001, GDPR, UK GDPR, and EU AI Act requirements for identity, infrastructure, applications, and internet-facing versus internal services.
- Added a `References` section to `docs/compliance/azure-compliance-baseline.md` with UK public-sector and legislation source URLs.
- Added `docs/compliance/azure-agentic-ai-security-baseline.md`, a draft Azure security baseline for agentic AI applications mapped to the OWASP Top 10 for Agentic Applications, covering identity, infrastructure, tools, memory, inter-agent communication, and containment controls.
- Added contents sections to both Azure compliance baseline documents for easier navigation across H1, H2, and H3 headings.

### Changed
- Improved `agentic-security-assessment/SKILL.md` for GPT-5.5-oriented assessment output with executive-first reporting, concern markers, evidence-strength labels, top-concern prioritisation, and concise standards/reference citations.
- Improved `azure-compliance-assessment/SKILL.md` for GPT-5.5-oriented assessment and proof outputs with executive-first reporting, concern markers, evidence-strength labels, top-concern or proven-control prioritisation, and concise standards/reference citations.
- Expanded `dotnet-agentic-security/references/data/ef-core-and-provider-switching.md` with project scaffolding commands, `dotnet ef` migration workflow, and seed data examples.
- Expanded `dotnet-agentic-security` testing guidance with programmatic `Testcontainers` examples and added library guidance for `Refit`, `FluentValidation`, and `Spectre.Console.Cli`.
- Hardened `agentic-security-assessment/SKILL.md` Mermaid guidance so generated diagrams use Mermaid-safe node labels, avoid raw bracket syntax in label text, and include a final parseability sanity check.

## [1.0.0] - 2026-01-04

### Added
- Added **Architecture Change Record (ACR)** skill for creating professional architecture change records
  - MADR format with status tracking and decision documentation
  - Automatic UML diagram generation using Mermaid (current and proposed architecture)
  - Before/after folder structure documentation with change annotations
  - Dual risk analysis (risks of making vs. not making the change) using 3-point scale
  - Automatic index file management with multiple views (by status, risk level, category)
  - Sequential numbering system (ACR-0001, ACR-0002, etc.)
  - Support for component, sequence, system context, and ER diagrams
  - Comprehensive implementation notes and rollback strategies
- Added **Changelog Updater** skill for maintaining professional changelogs
  - Automatic analysis of git commits since branch divergence
  - Smart categorization into Added, Changed, Fixed, Removed, Security, Deprecated
  - User-focused descriptions following Keep a Changelog format
  - Filtering of internal/non-user-facing changes
  - Breaking change detection and highlighting
  - Semantic versioning support
- Added **Documentation Site Creation** skill for building comprehensive Hugo-based documentation sites
  - Hugo static site generator with Lotus Docs theme integration
  - Custom homepage creation with hero sections, feature grids, and CTAs
  - Large documentation file restructuring with _index.md landing pages
  - GitHub Actions workflow for automated deployment
  - Custom domain configuration and DNS setup guidance
  - Screenshot strategy and placeholder management
  - Complete phase-by-phase methodology (Planning → Setup → Content → Deployment)
  - Support for both same-repository and separate documentation repositories
- Added **Meeting Notes Formatter** skill for transforming raw meeting notes into clean summaries
  - Automatic extraction of key decisions and action items
  - Owner assignment for action items
  - Discussion point summarization
  - Professional output formatting

[Unreleased]: https://github.com/garrardkitchen/claude-skills/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/garrardkitchen/claude-skills/releases/tag/v1.0.0
