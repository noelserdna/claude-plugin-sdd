---
name: setup
description: "Initializes the SDD pipeline in the current project. Creates pipeline-state.json, verifies the plugin is active, and upgrades hooks v1 to v2 if needed. Use when: (1) Setting up a new project for SDD pipeline, (2) Starting SDD on an existing codebase, (3) Verifying that SDD automation is properly installed, (4) Upgrading from hooks v1 to v2. Triggers: 'setup SDD', 'init pipeline', 'install SDD', 'start SDD project', 'iniciar SDD', 'configurar pipeline', 'initialize SDD', 'upgrade SDD'."
version: "2.1.0"
---

# SDD Setup

You are the **SDD Setup** initializer. Your job is to initialize the SDD pipeline in the current target project.

> **Note:** Since SDD is installed as a Claude Code plugin, hooks, agents, and settings are managed automatically by the plugin system. This skill handles project-specific initialization and upgrade detection.

## Prerequisites

- The current directory must be the **target project** (not the plugin repo itself)
- The `sdd` plugin must be installed (`/plugin list` should show `sdd`)

## Setup Process

### Step 0: Upgrade Detection

Before any initialization, check for existing v1 hooks that need migration:

1. Check `pipeline-state.json` for `hooksVersion` field
   - If `hooksVersion >= 2`: already upgraded, skip migration
   - If `pipeline-state.json` exists but `hooksVersion` is missing: v1 installation, needs upgrade
   - If `pipeline-state.json` does not exist: fresh install, proceed to Step 1
2. If v1 detected, check `.claude/settings.json` for v1 signals:
   - `PreToolUse` hook with matcher containing `SessionStart` (should be a `SessionStart` event)
   - `permissionDecision` without `hookSpecificOutput` wrapper in hook scripts
   - H4 stop hook with `echo {}` placeholder
   - Timeout values in milliseconds (>100) instead of seconds
3. If v1 detected:
   - Inform user: "Detected hooks v1 configuration — upgrading to v2"
   - Add `"sddVersion": "2.4.1"` and `"hooksVersion": 2` to `pipeline-state.json`
   - If `.claude/settings.json` has duplicate hooks already provided by the plugin (H1-H3, H5), remove them — the plugin `hooks.json` provides these automatically
   - Keep only project-specific optional hooks (H4, H6, H7) in `.claude/settings.json`
4. Report migration result

### Step 1: Verify Plugin is Active

1. Confirm the SDD plugin is loaded by checking that this skill was invocable
2. Verify current directory is a valid project (has `.git/` or user confirms)

### Step 2: Initialize Pipeline State

If `pipeline-state.json` does not exist:
- Create it with all stages set to `pending`:

```json
{
  "sddVersion": "2.4.1",
  "hooksVersion": 2,
  "currentStage": "requirements-engineer",
  "lastUpdated": "[NOW ISO-8601]",
  "stages": {
    "requirements-engineer":    { "status": "pending", "outputHash": null, "lastRun": null, "staleReason": null },
    "specifications-engineer":  { "status": "pending", "outputHash": null, "lastRun": null, "staleReason": null },
    "spec-auditor":             { "status": "pending", "outputHash": null, "lastRun": null, "staleReason": null },
    "test-planner":             { "status": "pending", "outputHash": null, "lastRun": null, "staleReason": null },
    "plan-architect":           { "status": "pending", "outputHash": null, "lastRun": null, "staleReason": null },
    "task-generator":           { "status": "pending", "outputHash": null, "lastRun": null, "staleReason": null },
    "task-implementer":         { "status": "pending", "outputHash": null, "lastRun": null, "staleReason": null }
  }
}
```

If `pipeline-state.json` already exists:
- **Ensure** `sddVersion` and `hooksVersion` fields are present (add them if missing — this is the v1→v2 upgrade)
- Preserve all existing pipeline state
- Report current state

### Step 3: Check Dependencies

Check if `jq` is available: `command -v jq`
- If not: warn user that hooks will fall back to `node -e` (slower but functional)

Check if `node` is available: `command -v node`
- If not: warn user that MCP server and context augmentation hook require Node.js 18+

### Step 3.5: Build MCP Server

Check if `server/dist/` exists in the plugin directory:
- If it exists: skip (already built)
- If not: run `cd server && npm install && npm run build`
- Report build status (success/failure/skipped)

### Step 3.6: Optional — Code Intelligence

Inform the user about the code intelligence feature:
- `/sdd:code-index` indexes the project codebase for deep traceability
- Works standalone but produces richer results with **GitNexus** installed
- Run after `/sdd:dashboard` to enable code-aware blast radius analysis
- This is optional and can be configured later

### Step 3.7: Optional — Dashboard Server & HTTP Hooks

Inform the user about real-time dashboard updates:
- The plugin includes a dashboard HTTP+SSE server at `server/dist/dashboard-entry.js`
- Start it with: `node <plugin-path>/server/dist/dashboard-entry.js`
- Set `SDD_DASHBOARD_PORT` env var for custom port (default: 3001)
- Dashboard available at `http://localhost:3001/`
- SSE stream at `http://localhost:3001/events`
- For HTTP hooks that POST events to the dashboard, add to `.claude/settings.json`:
  - SessionStart, PostToolUse, SubagentStart/Stop, TaskCompleted, Stop, SessionEnd hooks posting to `http://localhost:3001/hooks/*`
- These hooks are optional and silently fail if the dashboard server is not running

### Step 4: Verification

Report results:

```
## SDD Pipeline Initialized

| Component | Status |
|-----------|--------|
| Plugin: sdd | Active |
| Pipeline: pipeline-state.json | Initialized / Already exists (hooksVersion: 2) |
| Hooks version | v2 / Upgraded from v1 / Fresh install |
| MCP Server: server/dist/ | Built / NOT BUILT (run manually) |
| Dashboard Server | Available / Info provided |
| Dependency: jq | Available / MISSING (using node fallback) |
| Dependency: node | Available / MISSING (MCP server requires Node.js 18+) |

### What the Plugin Provides (automatic)
- Hook H1: Session start pipeline status injection (SessionStart event)
- Hook H2: Upstream artifact immutability guard (hookSpecificOutput format)
- Hook H3: Pipeline state auto-updater
- Hook H5: Context augmentation (injects SDD traceability into file reads)
- Agents A1-A8: Constitution enforcer, cross-auditor, context keeper, requirements watcher, spec compliance, test coverage, traceability validator, health monitor
- MCP Server: 5 tools (query, impact, context, coverage, trace) for live traceability queries

### Next Steps
1. Run `/sdd:pipeline-status` to verify pipeline state
2. Begin with `/sdd:requirements-engineer` for a new project
3. (Optional) Run `/sdd:code-index` after dashboard to enable code intelligence
4. (Optional) Start dashboard server for real-time updates
```

## Constraints

- Never overwrite existing pipeline-state.json stages (only add missing fields like sddVersion/hooksVersion)
- Always verify file integrity after creation
- Warn about missing `jq` but don't fail
