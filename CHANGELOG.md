# Changelog

All notable changes to the SDD plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-02-27

### Added
- New utility skill: `dashboard` — generates a visual HTML traceability dashboard from SDD pipeline artifacts
  - Scans all pipeline directories for artifact definitions (REQ, UC, WF, API, BDD, INV, ADR, NFR, RN, FASE, TASK)
  - Extracts cross-references and relationship types (implements, verifies, orchestrates, etc.)
  - Builds `dashboard/traceability-graph.json` with full artifact graph and statistics
  - Generates self-contained `dashboard/index.html` (no external dependencies)
  - Interactive features: pipeline status bar, traceability matrix, filters, detail panel
  - Extended ID patterns supporting compound IDs (REQ-EXT-001, INV-SYS-001, API-pdf-reader)
- Reference documents: `id-patterns-extended.md`, `graph-schema.md`, `html-template.md`
- New agent: Requirements Watcher (A4) — detects changes in requirements since last dashboard generation
- New agent: Spec Compliance Checker (A5) — verifies src/ implements what spec/ declares
- New agent: Test Coverage Monitor (A6) — calculates % of REQs with BDD/test coverage
- New agent: Traceability Validator (A7) — suspect link detection inspired by IBM DOORS
- New agent: Pipeline Health Monitor (A8) — health score 0-100 with actionable recommendations
- New utility skill: `sync-notion` — bidirectional sync of SDD artifacts with Notion databases
  - Push: creates/updates Notion databases with relations for REQ, UC, WF, API, BDD, INV, ADR, TASK
  - Pull: detects Notion changes and applies them to local markdown (with confirmation)
  - Rate-limited API calls (3 req/s), idempotent push, conflict resolution
  - Reference documents: `notion-schema.md`, `sync-protocol.md`

## [1.0.0] - 2026-02-07

### Added
- Initial plugin release migrated from sdd-skills repository
- 9 pipeline skills: requirements-engineer, specifications-engineer, spec-auditor, test-planner, plan-architect, task-generator, task-implementer, security-auditor, req-change
- 3 utility skills: pipeline-status, traceability-check, session-summary
- 1 setup skill: setup (initializes pipeline-state.json)
- 3 agents: constitution-enforcer (A1), cross-auditor (A2), context-keeper (A3)
- 4 hooks: session-start (H1), upstream-guard (H2), state-updater (H3), stop-hook (H4)
- SDD Constitution as shared reference
- All skills namespaced under `sdd:` (e.g., `/sdd:requirements-engineer`)
