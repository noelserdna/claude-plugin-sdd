# Notion Database Schema

Defines the Notion database structure for syncing SDD artifacts. Each artifact type maps to a Notion database with specific properties and relations.

## Parent Page Structure

```
SDD: {ProjectName}              ← Parent page (NOTION_PARENT_PAGE_ID)
  ├── Requirements DB            ← 1 database per artifact type
  ├── Use Cases DB
  ├── Workflows DB
  ├── API Contracts DB
  ├── BDD Scenarios DB
  ├── Invariants DB
  ├── ADR DB
  ├── Tasks DB
  └── Pipeline Status DB
```

## Database Definitions

### Requirements DB

| Property | Type | Description |
|----------|------|-------------|
| Name (title) | title | REQ ID + title (e.g., "REQ-EXT-001: Extract PDF text") |
| REQ ID | rich_text | The artifact ID (e.g., "REQ-EXT-001") |
| Category | select | EXT, CVA, F, NF, C, etc. |
| Priority | select | Must Have, Should Have, Could Have, Won't Have |
| Status | select | Full, Partial, None, Orphan |
| Source File | url | Relative path to definition file |
| Line | number | Line number in source file |
| Use Cases | relation | → Use Cases DB |
| BDD Scenarios | relation | → BDD Scenarios DB |
| Tasks | relation | → Tasks DB |

### Use Cases DB

| Property | Type | Description |
|----------|------|-------------|
| Name (title) | title | UC ID + title |
| UC ID | rich_text | The artifact ID |
| Source File | url | Relative path |
| Line | number | Line number |
| Requirements | relation | → Requirements DB |
| Workflows | relation | → Workflows DB |

### Workflows DB

| Property | Type | Description |
|----------|------|-------------|
| Name (title) | title | WF ID + title |
| WF ID | rich_text | The artifact ID |
| Source File | url | Relative path |
| Line | number | Line number |
| Use Cases | relation | → Use Cases DB |
| API Contracts | relation | → API Contracts DB |

### API Contracts DB

| Property | Type | Description |
|----------|------|-------------|
| Name (title) | title | API ID + title |
| API ID | rich_text | The artifact ID |
| Source File | url | Relative path |
| Line | number | Line number |
| Workflows | relation | → Workflows DB |
| BDD Scenarios | relation | → BDD Scenarios DB |

### BDD Scenarios DB

| Property | Type | Description |
|----------|------|-------------|
| Name (title) | title | BDD ID + title |
| BDD ID | rich_text | The artifact ID |
| Source File | url | Relative path |
| Line | number | Line number |
| Requirements | relation | → Requirements DB |
| API Contracts | relation | → API Contracts DB |

### Invariants DB

| Property | Type | Description |
|----------|------|-------------|
| Name (title) | title | INV ID + title |
| INV ID | rich_text | The artifact ID |
| Scope | select | SYS, SEC, BUS, etc. |
| Source File | url | Relative path |
| Line | number | Line number |

### ADR DB

| Property | Type | Description |
|----------|------|-------------|
| Name (title) | title | ADR ID + title |
| ADR ID | rich_text | The artifact ID |
| Decision Status | select | Proposed, Accepted, Deprecated, Superseded |
| Source File | url | Relative path |
| Line | number | Line number |

### Tasks DB

| Property | Type | Description |
|----------|------|-------------|
| Name (title) | title | TASK ID + description |
| Task ID | rich_text | The artifact ID |
| FASE | select | FASE-0, FASE-1, ..., FASE-N |
| Status | select | Pending, In Progress, Done |
| Commit Type | select | feat, fix, refactor, test, docs, chore |
| Requirements | relation | → Requirements DB |

### Pipeline Status DB

| Property | Type | Description |
|----------|------|-------------|
| Name (title) | title | Stage name |
| Status | select | done, stale, running, error, pending |
| Last Run | date | ISO-8601 timestamp |
| Artifact Count | number | Count of artifacts in stage |
| Order | number | Stage order (1-7) |

## Notion API Property Types Reference

| SDD Type | Notion Property Type | API Field |
|----------|---------------------|-----------|
| ID string | `rich_text` | `{ "rich_text": [{ "text": { "content": "value" } }] }` |
| Title | `title` | `{ "title": [{ "text": { "content": "value" } }] }` |
| Category/Status | `select` | `{ "select": { "name": "value" } }` |
| Line number | `number` | `{ "number": 42 }` |
| File path | `url` | `{ "url": "path/to/file.md" }` |
| Relation | `relation` | `{ "relation": [{ "id": "page-uuid" }] }` |
| Timestamp | `date` | `{ "date": { "start": "2026-02-27" } }` |

## Color Mapping for Select Options

| Status Value | Notion Color |
|-------------|-------------|
| done / Full / Must Have | green |
| running / Partial / Should Have | yellow |
| error / None | red |
| stale | orange |
| pending / Could Have | gray |
| Won't Have / Orphan | purple |
