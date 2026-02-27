---
name: dashboard
description: "Generates a visual HTML traceability dashboard from SDD pipeline artifacts.
  Scans all pipeline directories, extracts artifact IDs and cross-references,
  builds a structured JSON graph, and produces a self-contained HTML dashboard.
  Triggers: 'dashboard', 'visualize pipeline', 'traceability dashboard',
  'show traceability', 'generar dashboard', 'visualizar trazabilidad',
  'show dashboard', 'pipeline visualization', 'ver dashboard'."
version: "1.0.0"
---

# SDD Traceability Dashboard

You are the **SDD Dashboard Generator**. Your job is to scan all SDD pipeline artifacts, extract artifact definitions and cross-references, build a structured traceability graph, and generate a self-contained HTML dashboard that opens in the user's browser.

## Relationship to Other Skills

- **Complements** `traceability-check`: That skill produces a text report; this skill produces a visual, interactive HTML dashboard with the same underlying data.
- **Reads** output from ALL pipeline stages: `requirements/`, `spec/`, `audits/`, `test/`, `plan/`, `task/`, `src/`, `tests/`.
- **Reads** `pipeline-state.json` for stage status information.
- **Writes** to `dashboard/` only (never modifies pipeline artifacts).
- **Does NOT participate** in the linear pipeline chain — this is a utility skill.

## Output Artifacts

| File | Purpose |
|------|---------|
| `dashboard/traceability-graph.json` | Structured graph of all artifacts and relationships |
| `dashboard/index.html` | Self-contained HTML dashboard (CSS+JS inline) |

## Process

### Step 1: Read Pipeline State

Read `pipeline-state.json` from the project root.

- If it exists: extract `currentStage` and each stage's `status` and `lastRun`.
- If it does not exist: set all stages to `status: "unknown"` and `lastRun: null`.

Also determine the project name:
1. From `pipeline-state.json` project field, if present
2. From `package.json` `name` field, if present
3. From the current directory name as fallback

### Step 2: Discover Artifact Directories

Use Glob to check which of these directories exist and contain `.md` files:

| Directory | Pipeline Stage |
|-----------|---------------|
| `requirements/` | requirements-engineer |
| `spec/` | specifications-engineer |
| `audits/` | spec-auditor |
| `test/` | test-planner |
| `plan/` | plan-architect |
| `task/` | task-generator |
| `src/` | task-implementer |
| `tests/` | task-implementer |

Record which directories exist and which are empty. Report any missing directories as informational notes (not errors — partial pipelines are normal).

### Step 3: Extract Artifact Definitions

Scan each existing directory for artifact ID definitions using the patterns from `references/id-patterns-extended.md`.

For each artifact found, extract:
```json
{
  "id": "REQ-EXT-001",
  "type": "REQ",
  "category": "EXT",
  "title": "text from same line or next line after ID",
  "file": "requirements/REQUIREMENTS.md",
  "line": 42,
  "priority": "Must Have",
  "stage": "requirements-engineer"
}
```

**Extraction rules:**
- **ID**: Match using definition patterns (headings, table rows).
- **Type**: The prefix before the first hyphen (REQ, UC, WF, etc.).
- **Category**: The middle segment for compound IDs (EXT in REQ-EXT-001, SYS in INV-SYS-001), or null for simple IDs.
- **Title**: The text following the ID on the same line (after stripping markdown formatting). If the ID is alone on a heading line, use the next non-empty line.
- **File**: Relative path from project root using forward slashes.
- **Line**: 1-indexed line number where the ID is defined.
- **Priority**: Extract from adjacent table columns if present (look for "Priority", "Prioridad", "MoSCoW" column headers). Null if not in a table or no priority column.
- **Stage**: Mapped from the directory using the type-to-stage mapping in `references/id-patterns-extended.md`.

**Deduplication**: If the same ID appears in multiple files, use the first occurrence (by file path alphabetical order) as the definition.

### Step 4: Extract Relationships

Scan ALL markdown files in `requirements/`, `spec/`, `audits/`, `test/`, `plan/`, `task/` for cross-references between artifact IDs.

**Use the universal reference pattern** from `references/id-patterns-extended.md` to find all ID mentions. For each mention:
1. Determine the **source**: the artifact that "owns" the current file (e.g., if scanning `spec/use-cases/UC-001.md`, the source is `UC-001`).
2. Determine the **target**: the referenced ID.
3. Skip self-references (source === target).
4. Determine the **relationship type** by context:

| Context | Relationship Type |
|---------|-------------------|
| UC file referencing REQ | `implements` |
| WF file referencing API | `orchestrates` |
| BDD/test file referencing REQ or UC | `verifies` |
| INV referencing REQ | `guarantees` |
| ADR referencing REQ or NFR | `decides` |
| TASK referencing FASE | `decomposes` |
| `Refs:` field in task/commit | `implemented-by` |
| "Specs a Leer" / "Reads" section in FASE | `reads-from` |
| Any other cross-reference | `traces-to` |

**Range expansion**: When encountering range patterns like `INV-SEC-001..007`, expand to 7 individual references (INV-SEC-001 through INV-SEC-007).

**Deduplication**: Remove duplicate relationships (same source + target + type).

### Step 5: Build traceability-graph.json

Assemble the JSON structure following the schema in `references/graph-schema.md`:

1. **pipeline**: Merge stage statuses from Step 1 with artifact counts from Step 3.
2. **artifacts**: All extracted artifacts from Step 3.
3. **relationships**: All extracted relationships from Step 4.
4. **statistics**: Compute:
   - `totalArtifacts`: count of all artifacts
   - `byType`: count per artifact type
   - `totalRelationships`: count of all relationships
   - `traceabilityCoverage`:
     - `reqsWithUCs`: REQs that have at least one incoming `implements` relationship from a UC
     - `reqsWithBDD`: REQs that have at least one incoming `verifies` relationship from a BDD
     - `reqsWithTasks`: REQs traceable to at least one TASK (through any chain of relationships)
   - `orphans`: artifact IDs that have zero incoming relationships (no other artifact references them)
   - `brokenReferences`: IDs that appear in cross-references but are not defined in any artifact

Write the JSON to `dashboard/traceability-graph.json` with 2-space indentation.

### Step 6: Generate HTML Dashboard

1. Read the HTML template from `references/html-template.md` (extract the content inside the ```html code block).
2. Read the JSON from `dashboard/traceability-graph.json`.
3. Replace `{{DATA_JSON}}` with the raw JSON content.
4. Replace `{{PROJECT_NAME}}` with the project name.
5. Write the result to `dashboard/index.html`.

### Step 7: Open in Browser and Report

Execute the appropriate command to open the dashboard:
- **Windows**: `start dashboard/index.html`
- **macOS**: `open dashboard/index.html`
- **Linux**: `xdg-open dashboard/index.html`

Report a summary to the user:

```
## Dashboard Generated

| Metric | Value |
|--------|-------|
| Total Artifacts | {N} |
| Artifact Types | REQ:{n}, UC:{n}, WF:{n}, API:{n}, BDD:{n}, INV:{n}, ADR:{n}, TASK:{n} |
| Total Relationships | {N} |
| Traceability Coverage | {N}% REQs with UCs |
| Orphaned Artifacts | {N} |
| Broken References | {N} |
| Pipeline Stage | {currentStage} |

Files written:
- `dashboard/traceability-graph.json`
- `dashboard/index.html`

Dashboard opened in default browser.
```

If there are broken references or orphans, list the top 5 of each with file locations.

## Constraints

- **Write only to `dashboard/`**: Never modify pipeline artifacts (`requirements/`, `spec/`, `plan/`, `task/`, etc.).
- **Tolerate partial pipelines**: If only `requirements/` exists, still generate a dashboard with whatever is available.
- **No external dependencies**: The HTML file must be fully self-contained (no CDN links, no external CSS/JS).
- **Forward-slash paths**: Always use forward slashes in file paths within the JSON, even on Windows.
- **JSON validity**: The output JSON must be valid and parseable. Escape special characters in titles.
- **Idempotent**: Running the dashboard multiple times overwrites previous output without side effects.
- **Output Language**: Match the user's language for the summary report. Technical terms and artifact IDs remain in English.
