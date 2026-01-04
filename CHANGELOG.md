# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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

[Unreleased]: https://github.com/yourusername/claude-skills/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/yourusername/claude-skills/releases/tag/v1.0.0
