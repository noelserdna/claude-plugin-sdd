# Notion Sync Protocol

Defines the bidirectional synchronization algorithm between local SDD artifacts and Notion databases.

## Sync State File

The sync maintains state in `.sdd-notion-sync.json` at the project root:

```json
{
  "lastSync": "2026-02-27T15:30:00.000Z",
  "direction": "push",
  "notionParentPageId": "page-uuid",
  "databases": {
    "requirements": {
      "notionDbId": "db-uuid",
      "pages": {
        "REQ-EXT-001": {
          "notionPageId": "page-uuid",
          "localHash": "sha256:abc...",
          "notionLastEdited": "2026-02-27T15:30:00.000Z",
          "syncStatus": "synced"
        }
      }
    }
  }
}
```

### Sync Status Values

| Status | Meaning |
|--------|---------|
| `synced` | Local and Notion are in agreement |
| `local-ahead` | Local file changed since last sync |
| `notion-ahead` | Notion page edited since last sync |
| `conflict` | Both local and Notion changed since last sync |
| `new-local` | Exists locally but not in Notion |
| `new-notion` | Exists in Notion but not locally |
| `deleted-local` | Was synced but local artifact removed |

## Push Protocol (Local → Notion)

### Phase 1: Authentication

```bash
# Verify Notion API key
curl -s -H "Authorization: Bearer $NOTION_API_KEY" \
     -H "Notion-Version: 2022-06-28" \
     https://api.notion.com/v1/users/me
```

If authentication fails, report the error and stop.

### Phase 2: Database Setup

For each artifact type that has artifacts:

1. **Check if database exists** (from `.sdd-notion-sync.json`):
   - If yes, verify it still exists via API. If deleted, recreate.
   - If no, create a new database under the parent page.

2. **Create database** via POST `/v1/databases`:
   ```bash
   curl -s -X POST https://api.notion.com/v1/databases \
     -H "Authorization: Bearer $NOTION_API_KEY" \
     -H "Notion-Version: 2022-06-28" \
     -H "Content-Type: application/json" \
     -d '{
       "parent": { "type": "page_id", "page_id": "PARENT_PAGE_ID" },
       "title": [{ "text": { "content": "Requirements DB" } }],
       "properties": { ... }
     }'
   ```

3. Save database ID to sync state.

### Phase 3: Upsert Artifacts

For each artifact in `dashboard/traceability-graph.json`:

1. **Check sync state**: Does this artifact have a Notion page?
2. **Create** (POST `/v1/pages`) if new:
   ```bash
   curl -s -X POST https://api.notion.com/v1/pages \
     -H "Authorization: Bearer $NOTION_API_KEY" \
     -H "Notion-Version: 2022-06-28" \
     -H "Content-Type: application/json" \
     -d '{
       "parent": { "database_id": "DB_ID" },
       "properties": { ... }
     }'
   ```
3. **Update** (PATCH `/v1/pages/{page_id}`) if exists and local-ahead.
4. Save page ID and hash to sync state.

### Phase 4: Create Relations

After all pages are created, create Notion relations between databases:

1. For each relationship in the graph, find the source and target Notion page IDs.
2. Update the source page's relation property to include the target page ID.

**Important**: Relations require both databases to exist and both pages to be created first.

### Phase 5: Sync Pipeline Status

Upsert 7 rows in the Pipeline Status DB, one per stage.

## Pull Protocol (Notion → Local)

### Phase 1: Detect Notion Changes

Query each database for pages modified after `lastSync`:

```bash
curl -s -X POST "https://api.notion.com/v1/databases/$DB_ID/query" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "timestamp": "last_edited_time",
      "last_edited_time": { "after": "LAST_SYNC_ISO" }
    }
  }'
```

### Phase 2: Diff Analysis

For each changed Notion page:
1. Compare Notion property values against local artifact values.
2. Classify as: `notion-ahead`, `conflict`, or `synced` (if change was from our push).

### Phase 3: Apply Changes (with user confirmation)

For `notion-ahead` changes:
1. Show the user what will change in local markdown files.
2. Wait for confirmation.
3. Apply changes to the markdown file at the correct line.
4. Mark affected pipeline stages as `stale` in `pipeline-state.json`.

For `conflict` changes:
1. Show both local and Notion versions to the user.
2. Let the user choose: keep local, keep Notion, or merge manually.

### Phase 4: Update Sync State

Update `.sdd-notion-sync.json` with new hashes and timestamps.

## Rate Limiting

Notion API rate limit: **3 requests per second**.

The sync must:
- Insert a 350ms delay between API calls.
- Use batch operations where possible (Notion supports up to 100 items per query).
- Estimate time: 330 REQs ≈ 110 seconds for initial push.

## Error Handling

| Error | Action |
|-------|--------|
| 401 Unauthorized | Report invalid API key, stop |
| 404 Page/DB not found | Recreate and resync |
| 429 Rate limited | Wait for `Retry-After` header duration, then retry |
| 502/503 Server error | Retry up to 3 times with exponential backoff |
| Network error | Report and stop, sync state preserves progress |

## Conflict Resolution Rules

| Scenario | Default Action |
|----------|---------------|
| Title changed in Notion | Update local title |
| Priority changed in Notion | Update local priority |
| New artifact in Notion | Create local artifact (append to requirements file) |
| Artifact deleted in Notion | Flag for user review (never auto-delete locally) |
| Structural change (ID rename) | Flag as conflict, require manual resolution |
