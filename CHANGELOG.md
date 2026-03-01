# Changelog

All notable changes to the SDD plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.6.0] - 2026-03-01

### Added
- **4 Onboarding Skills** for adopting SDD in existing projects (brownfield, drift, migration scenarios)
  - **`onboarding`** (v1.0.0): Project detector and SDD adoption planner
    - 7-phase diagnostic: environment check, SDD artifact scan, non-SDD doc scan, code/test analysis, scenario classification, health score estimation, action plan generation
    - 8 scenarios: greenfield, brownfield bare, SDD drift, partial SDD, brownfield with docs, tests-as-spec, multi-team, fork/migration
    - Health score 0-100 with per-dimension breakdown (requirements, specs, tests, architecture, traceability, code quality, pipeline state)
    - Modes: default (full), `--quick`, `--reassess`
    - Output: `onboarding/ONBOARDING-REPORT.md`
    - References: `detection-matrix.md` (25+ signals, weighted classification matrix), `action-plan-templates.md` (8 scenario templates with effort/health projections)
  - **`reverse-engineer`** (v1.0.0): Code to SDD artifact generator
    - 10-phase process with 2 user checkpoints (after inventory/analysis, after artifact generation)
    - Generates complete SDD artifacts: requirements (EARS syntax), specs, test plan, architecture plan, retroactive tasks, findings report
    - Findings taxonomy with 7 markers: `[DEAD-CODE]`, `[TECH-DEBT]`, `[WORKAROUND]`, `[INFRASTRUCTURE]`, `[ORPHAN]`, `[INFERRED]`, `[IMPLICIT-RULE]`
    - Language-specific analysis patterns: TypeScript, Python, Rust, Go, Java
    - Modes: default (full), `--scope=paths`, `--inventory-only`, `--continue`, `--findings-only`
    - Output: `requirements/`, `spec/`, `test/`, `plan/`, `task/`, `findings/`, `reverse-engineering/`
    - References: `code-analysis-patterns.md`, `requirement-extraction-heuristics.md`, `findings-taxonomy.md`, `retroactive-task-template.md`
  - **`reconcile`** (v1.0.0): Spec-code drift detection and alignment
    - 8-phase process: context loading, code scan, spec-code comparison, divergence classification, reconciliation plan, user review, apply changes, pipeline state update
    - 6 divergence types: `NEW_FUNCTIONALITY` (auto), `REMOVED_FEATURE` (auto), `BEHAVIORAL_CHANGE` (ask), `REFACTORING` (auto), `BUG_OR_DEFECT` (ask), `AMBIGUOUS` (ask)
    - Automatic resolution for safe types; user decision for ambiguous cases
    - Modes: default (full), `--dry-run`, `--scope=paths`, `--code-wins`
    - Output: `reconciliation/RECONCILIATION-REPORT.md` + updated specs/requirements
    - References: `divergence-classification.md`, `reconciliation-strategies.md`, `reconciliation-report-template.md`
  - **`import`** (v1.0.0): External documentation to SDD format converter
    - 7-phase process: format detection, parse input, mapping preview, user confirmation, generate SDD artifacts, quality check, pipeline state update
    - 6 formats supported: Jira (JSON/CSV), OpenAPI/Swagger (YAML/JSON), Markdown, Notion (markdown/CSV), CSV, Excel (.xlsx)
    - Auto-detect format with content inspection fallback
    - EARS syntax conversion from user stories, imperative statements, conditionals, API descriptions
    - Modes: default (auto-detect), `--format=TYPE`, `--target=requirements|specs|both`, `--merge`
    - Output: `requirements/`, `spec/`, `import/IMPORT-REPORT.md`
    - References: `format-parsers.md`, `mapping-rules.md`, `import-report-template.md`

## [1.5.0] - 2026-02-28

### Added
- **Commit Traceability Integration**: commits are now a first-class link in the SDD traceability chain
  - Extended chain: `REQ → UC → WF → API → BDD → INV → ADR → TASK → COMMIT → CODE → TEST`
  - task-implementer Phase 7: SHA capture via `git rev-parse --short HEAD` after each atomic commit
  - task-implementer Phase 8: progress reports include SHA (`→ commit abc1234`)
  - task-implementer Phase 9: completion report includes full commit log table (Task/SHA/Message/Refs)
  - CHECK-C03: full implementation procedure (search by Task trailer → fallback subject → verify file scope → graceful degradation)
  - commit-conventions.md: new "SHA Capture & Traceability" section documenting the extended chain
  - Dashboard graph schema: `commitRefs[]` on artifacts, `implemented-by-commit` relationship type
  - Dashboard statistics: `commitStats` (totalCommits, commitsWithRefs, commitsWithTasks, uniqueTasksCovered) and `reqsWithCommits` coverage metric
  - Dashboard Step 5.5: "Scan Commit References" — scans git log for `Refs:` and `Task:` trailers, builds commitRef objects, propagates to REQs
  - Dashboard HTML: "Commits" and "Requirements with Commits" stat cards, progress bar in Summary view, commit stats in Code Coverage view
  - traceability-check Step 5: "Commit Chain Verification" — TASK→commit mapping, gap detection (tasks without commits, commits without refs, broken refs), commit coverage metric
  - traceability-patterns.md: TASK ID pattern (`TASK-F\d{1,2}-\d{3,4}`), "Commit Reference Patterns" section (Task/Refs trailers, git log extraction)
  - req-change Phase 2 step 7: "Commit Impact Analysis" — artifact→commits→files blast radius estimation via git log

### Changed
- Dashboard schema remains v2 (backward compatible): `commitRefs` defaults to `[]`, `commitStats` to zeros
- All git checks include graceful degradation (skip if not inside a git repository)
- Session Report table expanded with SHA and Commit Message columns

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
