# Traceability Graph JSON Schema

Schema for `dashboard/traceability-graph.json` — the structured representation of all SDD artifacts and their relationships.

## Full Schema

```json
{
  "$schema": "traceability-graph-v1",
  "generatedAt": "2026-02-27T15:30:00.000Z",
  "projectName": "my-project",

  "pipeline": {
    "currentStage": "task-generator",
    "stages": [
      {
        "name": "requirements-engineer",
        "status": "done",
        "lastRun": "2026-02-25T10:00:00.000Z",
        "artifactCount": 330
      },
      {
        "name": "specifications-engineer",
        "status": "done",
        "lastRun": "2026-02-25T14:00:00.000Z",
        "artifactCount": 150
      },
      {
        "name": "spec-auditor",
        "status": "done",
        "lastRun": "2026-02-26T09:00:00.000Z",
        "artifactCount": 5
      },
      {
        "name": "test-planner",
        "status": "done",
        "lastRun": "2026-02-26T11:00:00.000Z",
        "artifactCount": 20
      },
      {
        "name": "plan-architect",
        "status": "done",
        "lastRun": "2026-02-26T15:00:00.000Z",
        "artifactCount": 10
      },
      {
        "name": "task-generator",
        "status": "running",
        "lastRun": null,
        "artifactCount": 0
      },
      {
        "name": "task-implementer",
        "status": "pending",
        "lastRun": null,
        "artifactCount": 0
      }
    ]
  },

  "artifacts": [
    {
      "id": "REQ-EXT-001",
      "type": "REQ",
      "category": "EXT",
      "title": "Extract text from PDF files",
      "file": "requirements/REQUIREMENTS.md",
      "line": 42,
      "priority": "Must Have",
      "stage": "requirements-engineer"
    }
  ],

  "relationships": [
    {
      "source": "UC-001",
      "target": "REQ-EXT-001",
      "type": "implements",
      "sourceFile": "spec/use-cases/UC-001.md",
      "line": 15
    }
  ],

  "statistics": {
    "totalArtifacts": 800,
    "byType": {
      "REQ": 330,
      "UC": 41,
      "WF": 15,
      "API": 20,
      "BDD": 42,
      "INV": 55,
      "ADR": 12,
      "NFR": 30,
      "RN": 25,
      "FASE": 8,
      "TASK": 242
    },
    "totalRelationships": 1200,
    "traceabilityCoverage": {
      "reqsWithUCs": { "count": 280, "total": 330, "percentage": 84.8 },
      "reqsWithBDD": { "count": 200, "total": 330, "percentage": 60.6 },
      "reqsWithTasks": { "count": 250, "total": 330, "percentage": 75.8 }
    },
    "orphans": ["INV-SYS-042", "NFR-015"],
    "brokenReferences": [
      {
        "ref": "UC-099",
        "referencedIn": "spec/workflows/WF-003.md",
        "line": 15
      }
    ]
  }
}
```

## Field Definitions

### Root

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `$schema` | string | Yes | Always `"traceability-graph-v1"` |
| `generatedAt` | string (ISO-8601) | Yes | When the graph was generated |
| `projectName` | string | Yes | Name of the project (from `package.json`, directory name, or `pipeline-state.json`) |

### pipeline

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `currentStage` | string | Yes | Active stage name from `pipeline-state.json`, or `"unknown"` |
| `stages` | array | Yes | Ordered array of 7 pipeline stages |

### pipeline.stages[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Stage identifier (e.g., `"requirements-engineer"`) |
| `status` | enum | Yes | `"done"`, `"stale"`, `"running"`, `"error"`, `"pending"` |
| `lastRun` | string or null | Yes | ISO-8601 timestamp of last completion, null if never run |
| `artifactCount` | number | Yes | Count of artifacts produced by this stage |

### artifacts[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique artifact ID (e.g., `"REQ-EXT-001"`) |
| `type` | string | Yes | Artifact type: REQ, UC, WF, API, BDD, INV, ADR, NFR, RN, FASE, TASK |
| `category` | string or null | No | Sub-category (EXT, CVA, SYS, SEC, F, NF) or null |
| `title` | string | Yes | Artifact title or first line of definition |
| `file` | string | Yes | Relative file path where defined |
| `line` | number | Yes | Line number of definition |
| `priority` | string or null | No | Priority level if available |
| `stage` | string | Yes | Pipeline stage that owns this artifact |

### relationships[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source` | string | Yes | ID of the referencing artifact |
| `target` | string | Yes | ID of the referenced artifact |
| `type` | string | Yes | Relationship type (see below) |
| `sourceFile` | string | Yes | File where the reference was found |
| `line` | number | Yes | Line number of the reference |

### Relationship Types

| Type | Meaning | Typical Direction |
|------|---------|-------------------|
| `implements` | UC implements REQ | UC → REQ |
| `orchestrates` | WF orchestrates API | WF → API |
| `verifies` | BDD verifies REQ/UC | BDD → REQ/UC |
| `guarantees` | INV guarantees domain rule | INV → REQ |
| `decides` | ADR decides architecture | ADR → REQ/NFR |
| `decomposes` | TASK decomposes FASE | TASK → FASE |
| `implemented-by` | Task Refs field | TASK → UC/API/INV |
| `reads-from` | FASE reads spec | FASE → spec artifacts |
| `traces-to` | Generic cross-reference | any → any |

### statistics

| Field | Type | Description |
|-------|------|-------------|
| `totalArtifacts` | number | Total count of all artifacts |
| `byType` | object | Count per artifact type |
| `totalRelationships` | number | Total count of all relationships |
| `traceabilityCoverage` | object | Coverage metrics |
| `orphans` | array of strings | Artifact IDs with no incoming references |
| `brokenReferences` | array | References to undefined artifacts |

### traceabilityCoverage

| Field | Type | Description |
|-------|------|-------------|
| `reqsWithUCs` | coverage | REQs that have at least one UC |
| `reqsWithBDD` | coverage | REQs that have at least one BDD |
| `reqsWithTasks` | coverage | REQs traceable to at least one TASK |

Each coverage object: `{ "count": N, "total": M, "percentage": P }`

## Notes

- The `file` and `sourceFile` fields use forward-slash paths relative to the project root.
- `line` is 1-indexed.
- Duplicate relationships (same source+target+type) are deduplicated.
- Artifacts with the same ID but found in multiple files use the first occurrence as the definition.
