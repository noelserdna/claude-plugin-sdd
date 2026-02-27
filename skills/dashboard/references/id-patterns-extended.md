# Extended ID Patterns for Dashboard

Extended regex patterns for extracting artifact IDs from real SDD projects. Superset of `traceability-check/references/traceability-patterns.md` — covers compound IDs, named IDs, and range expansions.

## Definition Patterns (where IDs are defined)

| Type | Regex | Examples | Typical Files |
|------|-------|----------|---------------|
| REQ (simple) | `^#+\s*REQ-(\d{3,4})` | REQ-001, REQ-042 | `requirements/REQUIREMENTS.md` |
| REQ (categorized) | `^#+\s*REQ-([A-Z]{1,4})-(\d{3,4})` | REQ-EXT-001, REQ-CVA-002, REQ-F-001 | `requirements/REQUIREMENTS.md` |
| REQ (table) | `\|\s*REQ-([A-Z]{0,4}-?\d{3,4})\s*\|` | table cells | `requirements/`, `spec/` |
| UC | `^#+\s*UC-(\d{3,4})` | UC-001, UC-041 | `spec/use-cases/` |
| WF | `^#+\s*WF-(\d{3,4})` | WF-001, WF-015 | `spec/workflows/` |
| API (numeric) | `^#+\s*API-(\d{3,4})` | API-001, API-020 | `spec/contracts/` |
| API (named) | `^#+\s*API-([a-z][a-z0-9-]+)` | API-pdf-reader, API-auth-login | `spec/contracts/` |
| BDD (numeric) | `^#+\s*BDD-(\d{3,4})` or `Scenario:\s*BDD-(\d{3,4})` | BDD-001, BDD-042 | `spec/tests/`, `test/` |
| BDD (named) | `Scenario:\s*BDD-([a-z][a-z0-9-]+)` | BDD-extraction, BDD-login-flow | `spec/tests/`, `test/` |
| INV (simple) | `^#+\s*INV-(\d{3,4})` | INV-001, INV-015 | `spec/domain/` |
| INV (scoped) | `^#+\s*INV-([A-Z]{2,6})-(\d{3,4})` | INV-SYS-001, INV-SEC-007 | `spec/domain/` |
| INV (table) | `\|\s*INV-([A-Z]{0,6}-?\d{3,4})\s*\|` | table cells | `spec/` |
| ADR | `^#+\s*ADR-(\d{3,4})` or filename `ADR-(\d{3,4})` | ADR-001 | `spec/adr/` |
| NFR | `^#+\s*NFR-(\d{3,4})` or `\|\s*NFR-(\d{3,4})\s*\|` | NFR-001 | `spec/nfr/` |
| RN | `^#+\s*RN-(\d{3,4})` or `\|\s*RN-(\d{3,4})\s*\|` | RN-001 | `spec/` |
| FASE | `^#+\s*FASE-(\d{1,2})` or filename `FASE-(\d{1,2})` | FASE-0, FASE-3 | `plan/fases/` |
| TASK | `^#+\s*TASK-F(\d{1,2})-(\d{3,4})` | TASK-F0-001, TASK-F2-012 | `task/` |

## Reference Pattern (Universal)

Single regex to match **any** ID reference in running text:

```regex
(REQ|UC|WF|API|BDD|INV|ADR|NFR|RN|FASE|TASK)[-‑](?:[A-Z]{0,6}[-‑])?(?:[a-z][a-z0-9-]*|\d{1,4})(?:[-‑]\d{3,4})?
```

### Breakdown

| Component | Matches |
|-----------|---------|
| `(REQ\|UC\|...)` | Type prefix |
| `[-‑]` | Hyphen or non-breaking hyphen |
| `(?:[A-Z]{0,6}[-‑])?` | Optional category (EXT, SYS, SEC, CVA, F, NF) |
| `(?:[a-z][a-z0-9-]*\|\d{1,4})` | Named ID (pdf-reader) or numeric (001) |
| `(?:[-‑]\d{3,4})?` | Optional sub-number (for TASK-F0-001) |

## Range Expansion

Some documents use range notation. The dashboard must expand these:

| Pattern | Example | Expansion |
|---------|---------|-----------|
| `{ID}..{ID}` | `INV-SEC-001..007` | INV-SEC-001 through INV-SEC-007 |
| `{ID}-{ID}` in lists | `REQ-001-005` | REQ-001 through REQ-005 (only when in range context) |
| `{ID}, {ID}, {ID}` | `UC-001, UC-003, UC-005` | Three separate references |

## Type-to-Stage Mapping

| Artifact Type | Pipeline Stage | Directory |
|---------------|---------------|-----------|
| REQ | requirements-engineer | `requirements/` |
| UC, WF, API, BDD, INV, ADR, NFR, RN | specifications-engineer | `spec/` |
| AUDIT | spec-auditor | `audits/` |
| TEST | test-planner | `test/` |
| FASE | plan-architect | `plan/` |
| TASK | task-generator | `task/` |
| (code) | task-implementer | `src/`, `tests/` |

## Priority Extraction

When found in tables, extract priority from adjacent columns:

| Column Headers | Values |
|----------------|--------|
| Priority, Prioridad | Must Have, Should Have, Could Have, Won't Have |
| MoSCoW | M, S, C, W |
| Level, Nivel | Critical, High, Medium, Low |
