---
name: setup
description: "Initializes the SDD pipeline in the current project. Creates pipeline-state.json and verifies the plugin is active. Use when setting up a new project for SDD pipeline."
version: "2.0.0"
---

# SDD Setup

You are the **SDD Setup** initializer. Your job is to initialize the SDD pipeline in the current target project.

> **Note:** Since SDD is installed as a Claude Code plugin, hooks, agents, and settings are managed automatically by the plugin system. This skill only handles project-specific initialization.

## Prerequisites

- The current directory must be the **target project** (not the plugin repo itself)
- The `sdd` plugin must be installed (`/plugin list` should show `sdd`)

## Setup Process

### Step 1: Verify Plugin is Active

1. Confirm the SDD plugin is loaded by checking that this skill was invocable
2. Verify current directory is a valid project (has `.git/` or user confirms)

### Step 2: Initialize Pipeline State

If `pipeline-state.json` does not exist:
- Create it with all stages set to `pending`:

```json
{
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
- Leave it untouched (preserve existing pipeline state)
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

### Step 4: Verification

Report results:

```
## SDD Pipeline Initialized

| Component | Status |
|-----------|--------|
| Plugin: sdd | Active |
| Pipeline: pipeline-state.json | Initialized / Already exists |
| MCP Server: server/dist/ | Built / NOT BUILT (run manually) |
| Dependency: jq | Available / MISSING (using node fallback) |
| Dependency: node | Available / MISSING (MCP server requires Node.js 18+) |

### What the Plugin Provides (automatic)
- Hook H1: Session start pipeline status injection
- Hook H2: Upstream artifact immutability guard
- Hook H3: Pipeline state auto-updater
- Hook H4: Stop hook pipeline consistency check
- Hook H5: Context augmentation (injects SDD traceability into file reads)
- Agent A1: Constitution enforcer
- Agent A2: Cross-auditor
- Agent A3: Context keeper
- Agent A4-A8: Requirements watcher, spec compliance, test coverage, traceability validator, health monitor
- MCP Server: 5 tools (query, impact, context, coverage, trace) for live traceability queries

### Next Steps
1. Run `/sdd:pipeline-status` to verify pipeline state
2. Begin with `/sdd:requirements-engineer` for a new project
3. (Optional) Run `/sdd:code-index` after dashboard to enable code intelligence
```

## Constraints

- Never overwrite existing pipeline-state.json
- Always verify file integrity after creation
- Warn about missing `jq` but don't fail
