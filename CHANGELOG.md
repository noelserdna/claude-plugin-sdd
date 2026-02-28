# Changelog

All notable changes to the SDD plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.4.0] - 2026-02-28

### Added
- **SDD System Guide**: self-contained HTML documentation page (`guide.html`) generated alongside the dashboard
  - Part 1: Full SDD system documentation (9 pipeline skills, traceability chain, automation hooks/agents, utility skills)
  - Part 2: Dashboard interpretation guide (all views, metrics, health score formula, color legend, glossary)
  - Sticky sidebar navigation with scroll-spy active tracking
  - Same dark theme as dashboard, responsive at 768px
  - "Guide" button in dashboard header linking to `guide.html`; "Back to Dashboard" link in guide
  - New reference: `guide-template.md`
- **Per-file Test Coverage Map** across pipeline skills
  - plan-architect: Coverage Map §7.4 in FASE templates (source file → test file mapping with classification)
  - task-generator: 1 test task per source file from Coverage Map; new validations V-13, V-14
  - task-implementer: per-file coverage verification (0% = CRITICAL, <80% domain logic = WARNING), coverage in completion report
  - New verification dimension: Dimension 4 (Coverage) with CHECK-COV-01 through CHECK-COV-04

### Changed
- Dashboard SKILL.md Step 9 now generates both `index.html` and `guide.html`
- Dashboard output artifacts: added `dashboard/guide.html`

## [1.3.0] - 2026-02-28

### Changed
- **Dashboard v3.0.0 — Comprehension Dashboard**: UX overhaul for non-technical stakeholders
  - New Executive Summary view as default tab: coverage progress bars, top gaps, artifact breakdown, pipeline status
  - Health Score hero banner: weighted letter grade (A-F) with actionable recommendations
  - Humanized all labels: UC → Use Cases, WF → Workflows, BDD → Acceptance Tests, INV → Business Rules, ADR → Decisions
  - Status labels: Full → Complete, Partial → In Progress, Spec Only → Specified, Untraced → Not Started
  - Color legend below stats cards for status dot meanings
  - WCAG AA contrast fixes: improved --text2, --text3, --yellow, --gray color values
  - Contextual tooltips on all stats cards and zero-count matrix cells
  - Responsive hero banner (column layout on mobile)
  - No schema changes — all improvements are UI/presentation only

## [1.2.0] - 2026-02-27

### Changed
- **Dashboard v2.0.0 — Comprehension Dashboard**: Major upgrade to the traceability dashboard skill
  - Traces REQs to source code: scans `src/` for `Refs:` comments in JSDoc, inline comments, and decorators
  - Traces REQs to tests: scans `tests/` for artifact references in test descriptions and file headers
  - Auto-classifies REQs by business domain (from prefix), technical layer (from FASE), and functional category (from section headers)
  - New graph schema v2: `codeRefs[]`, `testRefs[]`, `classification{}` on artifacts; `codeStats`, `testStats`, `classificationStats` in statistics
  - New relationship types: `implemented-by-code` (code→artifact), `tested-by` (test→artifact)
  - New HTML dashboard views: Matrix (enhanced), Classification (domain grouping), Code Coverage (file-level detail)
  - Redesigned detail panel with 5 tabs: Story (narrative), Trace Chain, Code, Tests, Documents
  - New filters: Domain, Layer, Category dropdowns
  - New status calculation: Full (REQ+UC+BDD+TASK+Code+Tests), Partial, Spec Only, Untraced
  - Symbol extraction from code: functions, classes, consts with fallback to filename:line
  - Test framework detection: vitest, jest, pytest, jasmine
  - Reference documents updated: `id-patterns-extended.md`, `graph-schema.md`, `html-template.md`

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
