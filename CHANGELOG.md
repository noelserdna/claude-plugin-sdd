# Changelog

All notable changes to the SDD plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
