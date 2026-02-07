# SDD Plugin for Claude Code

> **[Leer en español](README.es.md)**

Specification-Driven Development pipeline based on SWEBOK v4. A complete requirements-to-code pipeline with 13 skills, automated guardrails, and traceability enforcement.

## Installation

```
/plugin install github:noelserdna/claude-plugin-sdd
```

## Quick Start

```
/sdd:setup                       # Initialize pipeline in your project
/sdd:requirements-engineer       # Start gathering requirements
/sdd:specifications-engineer     # Transform requirements into specs
/sdd:spec-auditor                # Audit specs for defects
/sdd:test-planner                # Generate test strategy
/sdd:plan-architect              # Generate implementation plan
/sdd:task-generator              # Generate atomic tasks
/sdd:task-implementer            # Implement code from tasks
```

## Pipeline

```
requirements-engineer  ->  requirements/REQUIREMENTS.md
        |
specifications-engineer  ->  spec/
        |
spec-auditor (Audit)  ->  audits/AUDIT-BASELINE.md
        |
spec-auditor (Fix)  ->  corrected spec/
        |
test-planner  ->  test/TEST-PLAN.md, TEST-MATRIX-*.md, PERF-SCENARIOS.md
        |
plan-architect  ->  plan/ (FASE files, PLAN.md, ARCHITECTURE.md)
        |
task-generator  ->  task/TASK-FASE-*.md
        |
task-implementer  ->  src/, tests/, git commits
```

## Skills Reference

### Pipeline Skills (9)

| Skill | Command | Purpose |
|-------|---------|---------|
| Requirements Engineer | `/sdd:requirements-engineer` | Elicit, audit, and write requirements (SWEBOK Ch01) |
| Specifications Engineer | `/sdd:specifications-engineer` | Transform requirements into formal specs |
| Spec Auditor | `/sdd:spec-auditor` | Audit specs for defects; fix mode for corrections |
| Test Planner | `/sdd:test-planner` | Generate test strategy, matrices, perf scenarios (SWEBOK Ch04) |
| Plan Architect | `/sdd:plan-architect` | Generate FASE files and implementation plans |
| Task Generator | `/sdd:task-generator` | Decompose FASEs into atomic, reversible tasks |
| Task Implementer | `/sdd:task-implementer` | Implement code with TDD, atomic commits |
| Security Auditor | `/sdd:security-auditor` | OWASP/CWE security posture audit (lateral) |
| Req Change | `/sdd:req-change` | Manage requirement changes with pipeline cascade (lateral) |

### Utility Skills (3)

| Skill | Command | Purpose |
|-------|---------|---------|
| Pipeline Status | `/sdd:pipeline-status` | Show pipeline state, staleness, next action |
| Traceability Check | `/sdd:traceability-check` | Verify REQ-UC-WF-API-BDD-INV-ADR chain |
| Session Summary | `/sdd:session-summary` | Summarize session decisions and progress |

### Setup Skill (1)

| Skill | Command | Purpose |
|-------|---------|---------|
| Setup | `/sdd:setup` | Initialize pipeline-state.json in target project |

## Automation

The plugin automatically installs:

### Hooks

| Hook | Event | Purpose |
|------|-------|---------|
| H1 | PreToolUse (SessionStart) | Injects pipeline status into session context |
| H2 | PreToolUse (Edit/Write) | Blocks downstream skills from modifying upstream artifacts |
| H3 | PostToolUse (Write) | Auto-updates pipeline-state.json on artifact writes |
| H4 | Stop | Verifies pipeline state consistency on session end |

### Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| Constitution Enforcer (A1) | haiku | Validates operations against 11 SDD Constitution articles |
| Cross-Auditor (A2) | sonnet | Cross-references skill definitions for mismatches |
| Context Keeper (A3) | haiku | Maintains informal project context |

## Traceability Chain

Every artifact traces through the full chain:

```
REQ <> UC <> WF <> API <> BDD <> INV <> ADR <> RN
```

## Key Conventions

- **EARS syntax** for requirements: `WHEN <trigger> THE <system> SHALL <behavior>`
- **1 task = 1 commit** using Conventional Commits with `Refs:` and `Task:` trailers
- **Baseline auditing**: first audit creates baseline; subsequent audits report only new/regression findings
- **Clarification-first**: skills never assume, always ask with structured options

## Standards Referenced

- SWEBOK v4
- OWASP ASVS v4
- CWE
- IEEE 830
- ISO 14764
- C4 Model
- Gherkin/BDD

## License

MIT
