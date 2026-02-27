---
name: sync-notion
description: "Syncs SDD pipeline artifacts bidirectionally with Notion databases.
  Pushes requirements, use cases, tasks, and pipeline status to structured Notion DBs
  with relations. Pulls changes from Notion back to local markdown files.
  Requires NOTION_API_KEY and NOTION_PARENT_PAGE_ID environment variables.
  Triggers: 'sync notion', 'notion sync', 'push to notion', 'pull from notion',
  'sincronizar notion', 'enviar a notion'."
version: "1.0.0"
---

# SDD Notion Sync

You are the **SDD Notion Sync** skill. Your job is to synchronize SDD pipeline artifacts bidirectionally with Notion databases, creating a navigable, relational view of the entire traceability chain in Notion.

## Prerequisites

The user must set two environment variables:

| Variable | Purpose | Example |
|----------|---------|---------|
| `NOTION_API_KEY` | Notion integration API key | `ntn_xxxxxxxxxxxxx` |
| `NOTION_PARENT_PAGE_ID` | Page ID where databases will be created | `12345678-abcd-...` |

If either is missing, show setup instructions and stop.

## Relationship to Other Skills

- **Depends on** `dashboard`: Reads `dashboard/traceability-graph.json` for artifact and relationship data. If it doesn't exist, recommend running `/sdd:dashboard` first.
- **Reads** `pipeline-state.json` for stage statuses.
- **Writes** `.sdd-notion-sync.json` to track sync state.
- **May modify** local markdown files during pull (Notion → Local), marking affected stages as stale.

## Modes of Operation

### Mode 1: Push (Local → Notion) — Default

Push all local artifacts to Notion databases.

### Mode 2: Pull (Notion → Local)

Detect changes made in Notion and apply them to local markdown files.

### Mode 3: Status

Show sync state without making changes.

## Process

### Step 1: Verify Authentication

Check that `NOTION_API_KEY` and `NOTION_PARENT_PAGE_ID` are set:

```bash
if [ -z "$NOTION_API_KEY" ]; then
  echo "ERROR: NOTION_API_KEY not set"
  exit 1
fi
```

Verify the API key works by calling the Notion users/me endpoint:

```bash
curl -s -H "Authorization: Bearer $NOTION_API_KEY" \
     -H "Notion-Version: 2022-06-28" \
     https://api.notion.com/v1/users/me
```

If authentication fails, show the user how to create a Notion integration and set the env var.

### Step 2: Load Traceability Graph

Read `dashboard/traceability-graph.json`.

- If it does not exist, inform the user: "Run `/sdd:dashboard` first to generate the traceability graph."
- Extract all artifacts and relationships.

### Step 3: Load Sync State

Read `.sdd-notion-sync.json` from the project root.

- If it does not exist, this is a fresh sync (all artifacts are `new-local`).
- If it exists, load previous database IDs and page mappings.

### Step 4: Create/Verify Databases (Push mode)

For each artifact type that has artifacts, ensure a Notion database exists:

1. Check if the database ID from sync state is still valid (GET `/v1/databases/{id}`).
2. If missing or deleted, create a new database under the parent page using the schema from `references/notion-schema.md`.
3. Update sync state with database IDs.

Database creation order (to set up relations correctly):
1. Requirements DB
2. Use Cases DB
3. Workflows DB
4. API Contracts DB
5. BDD Scenarios DB
6. Invariants DB
7. ADR DB
8. Tasks DB
9. Pipeline Status DB

### Step 5: Upsert Artifacts (Push mode)

For each artifact in the traceability graph:

1. Check sync state for existing Notion page.
2. If new: CREATE page via POST `/v1/pages`.
3. If exists and local content changed (hash mismatch): UPDATE page via PATCH `/v1/pages/{id}`.
4. If exists and unchanged: SKIP.

**Rate limiting**: Insert 350ms delay between API calls to respect Notion's 3 req/s limit.

Report progress to the user:
```
Syncing: [████████░░░░░░░░░░░░] 120/330 REQs...
```

### Step 6: Create Relations (Push mode)

After all pages are created, create Notion relations:

1. For each relationship in the graph, look up source and target Notion page IDs from sync state.
2. Update the source page's relation property to include the target.
3. Skip relations where either page doesn't exist in Notion.

### Step 7: Detect Notion Changes (Pull mode)

Query each database for pages modified after `lastSync`:

1. For each changed page, compare property values against local artifact.
2. Classify changes as: `notion-ahead`, `conflict`, or `synced`.
3. Present changes to the user for confirmation before applying.

### Step 8: Apply Changes (Pull mode, with confirmation)

For confirmed changes:

1. Locate the local markdown file and line number.
2. Apply the change (title update, priority change, etc.).
3. Mark affected pipeline stages as `stale` in `pipeline-state.json`.

For conflicts, present both versions and let the user decide.

### Step 9: Update Sync State

Write updated `.sdd-notion-sync.json` with:
- New `lastSync` timestamp
- Updated page IDs and hashes
- Direction of last sync

### Step 10: Report Summary

```
## Notion Sync Complete

| Metric | Value |
|--------|-------|
| Direction | Push / Pull |
| Databases | {N} active |
| Pages Created | {N} |
| Pages Updated | {N} |
| Pages Unchanged | {N} |
| Relations Created | {N} |
| Duration | {N}s |

Notion workspace: {link to parent page}
```

## Constraints

- **Never store API keys**: Do not write `NOTION_API_KEY` to any file. Read from environment only.
- **Rate limiting**: Always respect Notion's 3 req/s limit with 350ms delays.
- **Confirmation for pulls**: Never auto-apply Notion → Local changes without user confirmation.
- **Never auto-delete**: If an artifact is deleted in Notion, flag it for user review rather than deleting locally.
- **Idempotent pushes**: Running push multiple times should not create duplicate pages.
- **JSON escape**: Properly escape special characters in artifact titles when building API payloads.
- **Output Language**: Match the user's language for reports. API calls and technical fields stay in English.
