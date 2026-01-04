# Architecture Change Records (ACRs)

This directory contains all Architecture Change Records for the Claude Code Skills Library.

## Overview

Architecture Change Records (ACRs) document significant architectural decisions and changes to the skills library and development workflow. Each record includes context, alternatives considered, decision rationale, risk analysis, and implementation guidance.

---

## Active Records

| ID | Title | Status | Date | Risk Level | Description |
|----|-------|--------|------|------------|-------------|
| [ACR-0001](0001-introduce-structured-skills-library.md) | Introduce Structured Skills Library | Accepted | 2026-01-04 | Low | Introduction of centralized skills repository for development workflow standardization |

---

## By Status

### 🟢 Accepted (Implemented)
- [ACR-0001: Introduce Structured Skills Library](0001-introduce-structured-skills-library.md) - Initial skills framework implementation

### 🟡 Accepted (Pending Implementation)
- None

### 🔵 Proposed (Under Review)
- None

### 🔴 Rejected
- None

### ⚫ Superseded
- None

### 🟤 Deprecated
- None

---

## By Risk Level

### High Risk Changes
- None

### Medium Risk Changes
- None

### Low Risk Changes
- [ACR-0001: Introduce Structured Skills Library](0001-introduce-structured-skills-library.md) - Accepted

---

## By Category

### Development Workflow
- [ACR-0001: Introduce Structured Skills Library](0001-introduce-structured-skills-library.md)

---

## Statistics

- **Total ACRs:** 1
- **Accepted (Implemented):** 1
- **Accepted (Pending):** 0
- **Proposed:** 0
- **Rejected:** 0
- **Superseded:** 0

---

## How to Create a New ACR

### Option 1: Use the Architecture Change Record Skill

If the ACR skill is registered in your Claude Code configuration:

```bash
# Invoke the skill
claude "Create an ACR for [your architectural change]"
```

### Option 2: Manual Creation

1. Copy the template from [TEMPLATE.md](TEMPLATE.md)
2. Determine the next sequential number (e.g., if last is 0001, use 0002)
3. Create file: `NNNN-descriptive-kebab-case-title.md`
4. Fill in all sections following MADR format
5. Add entry to this index file (README.md)
6. Submit for team review via pull request

---

## Guidelines

### When to Create an ACR

Create an ACR when you:

- Make significant architectural decisions affecting project structure
- Introduce new development patterns or frameworks
- Change fundamental workflows or processes
- Add/remove major dependencies or tools
- Restructure project organization
- Make decisions with long-term implications

### When NOT to Create an ACR

Skip ACRs for:

- Minor bug fixes or code refactoring
- Routine maintenance tasks
- Trivial configuration changes
- Individual feature implementations (unless architecturally significant)
- Experimental/POC work (create ACR if it becomes permanent)

### ACR Best Practices

1. **Create before implementation** - Document decision rationale while context is fresh
2. **Get team review** - ACRs benefit from diverse perspectives
3. **Update status** - Mark as Accepted → Implemented as work progresses
4. **Link related ACRs** - Show architectural evolution over time
5. **Be honest** - Document real constraints and trade-offs, not idealized scenarios
6. **Include risks for BOTH scenarios** - Analyze making AND not making the change
7. **Use diagrams** - Visual representations clarify complex architectures
8. **Document structure changes** - Show before/after folder organization
9. **Keep it updated** - Update ACR if understanding evolves during implementation
10. **Reference in commits** - Link to ACR in commit messages for traceability

### Review Process

1. **Author creates ACR** with Status: Proposed
2. **Submit PR** for team review
3. **Team discusses** - Address questions and concerns
4. **Consensus reached** - Update Status to Accepted or Rejected
5. **Implementation begins** - Reference ACR in related commits/PRs
6. **Mark Implemented** - Update status when change is complete
7. **Periodic review** - Revisit ACRs to ensure they reflect reality

---

## Template

See [TEMPLATE.md](TEMPLATE.md) for the complete ACR template structure including:

- MADR format sections
- Mermaid diagram examples
- Risk analysis tables
- Folder structure templates
- All required and optional sections

---

## Document History

| Date | Change | Author |
|------|--------|--------|
| 2026-01-04 | Created ACR index and added ACR-0001 | garrardkitchen|

---

**Last Updated:** 2026-01-04
**Next Review:** 2026-04-04
