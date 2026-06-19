---
trigger: always_on
description: >-
---


# AgentRecall v3.4.22 — Usage Guide

AgentRecall is a persistent memory system. Default surface: **5 tools** (two verbs + three essentials). Full surface: 18 tools via `npx agent-recall-mcp --full`. This guide describes how and when to use them.

**Two-verb model:** `session_start` (inhale — load context) and `session_end` (exhale — save and compound). Everything else is available but secondary; most agents never need more than the default 5. See [Automaticity Law](#why-5-default-tools) below.

## Setup

AgentRecall requires the MCP server to be running. If tool calls fail with "unknown tool", the human needs to install it first.

### Installation (human runs once)

**Claude Code:**
```bash
claude mcp add --scope user agent-recall -- npx -y agent-recall-mcp
```

**Cursor** (`.cursor/mcp.json`):
```json
{ "mcpServers": { "agent-recall": { "command": "npx", "args": ["-y", "agent-recall-mcp"] } } }
```

**VS Code / GitHub Copilot** (`.vscode/mcp.json`):
```json
{ "servers": { "agent-recall": { "command": "npx", "args": ["-y", "agent-recall-mcp"] } } }
```

**Windsurf** (`~/.codeium/windsurf/mcp_config.json`):
```json
{ "mcpServers": { "agent-recall": { "command": "npx", "args": ["-y", "agent-recall-mcp"] } } }
```

**Codex:**
```bash
codex mcp add agent-recall -- npx -y agent-recall-mcp
```

**Hermes Agent** (`~/.hermes/config.yaml`):
```yaml
mcp_servers:
  agent-recall:
    command: npx
    args: ["-y", "agent-recall-mcp"]
```

**Roo Code** (`.roo/mcp.json`):
```json
{ "mcpServers": { "agent-recall": { "command": "npx", "args": ["-y", "agent-recall-mcp"] } } }
```

**Any MCP-compatible agent:**
```
command: npx
args: ["-y", "agent-recall-mcp"]
transport: stdio
```

---

## Tools

AgentRecall's default surface provides **5 tools**. Start the server with `--full` to enable the complete 18-tool surface.

**Default tools (always available):** `session_start`, `session_end`, `remember`, `recall`, `check`

**Full-mode only (`--full`):** `memory_query`, `check_action`, `register_rule`, `pipeline_open`, `pipeline_close`, `pipeline_list`, `pipeline_current`, `pipeline_show`, `skill_write`, `skill_recall`, `skill_list`, `dashboard_export`, `session_end_reflect`, `project_board`, `project_status`, `digest`, `bootstrap_scan`, `bootstrap_import`

---

### Default tools

### `session_start`

**When:** Beginning of a session, to load prior context.

**What it returns:**
- `project` — detected project name
- `identity` — who the user is (1-2 lines)
- `insights` — top 5 awareness insights (title + confirmation count + severity)
- `active_rooms` — top 5 palace rooms by salience (with staleness flag + last_updated)
  _(Palace = your project's long-term knowledge store, organized into topic rooms like "architecture", "goals", "blockers". Salience = relevance score 0-1 based on recency, access frequency, and connections. Rooms with stale=true haven't been updated in 7+ days.)_
- `cross_project` — insights from other projects matching current context
- `recent` — today/yesterday journal briefs
- `watch_for` — predictive warnings from past correction patterns + decision calibration
- `corrections` — P0 behavioral rules (max 10, always loaded, never expire)
- `resume` — structured re-entry briefing: `last_date`, `last_trajectory`, `sessions_count`

**How to use the response:**
1. Read `identity` to calibrate your tone and approach
2. Read `insights` — these are battle-tested lessons. Follow them.
3. Read `watch_for` — these are patterns where you've been wrong before on this project. Adjust your approach.
4. Read `recent` to understand where the last session left off
5. Present a brief to the human: project name, last session summary, relevant insights

**Example call:**
```
session_start({ project: "auto" })
```

### `remember`

**When:** You learn something worth keeping. A decision, a bug fix, an insight, a session note.

**What it does:** Auto-classifies your content and routes it to the right store:
- Bug fix / lesson → knowledge store
- Architecture / decision → palace room
- Cross-project pattern → awareness system
- Session activity → journal

You do NOT need to decide where it goes. Just describe what to remember.

**How to use:**
```
remember({
  content: "We decided to use GraphQL instead of REST because the frontend needs flexible queries",
  context: "architecture decision"    // optional hint, improves routing
})
```

**Returns:** `routed_to` (which store), `classification` (content type), `auto_name` (semantic slug generated)

### `recall`

**When:** You need to find something from past sessions. A decision, a pattern, a lesson.

**What it does:** Searches ALL stores at once using Reciprocal Rank Fusion (RRF) — each source (palace, journal, insights) ranks internally, then positions merge so no single source dominates. Journal entries decay fast via Ebbinghaus curve (S=2 days); palace entries are near-permanent (S=9999). Returns ranked results with stable IDs.

**How to use:**
```
recall({ query: "authentication design", limit: 5 })
```

**Feedback:** After using results, rate them. Ratings use a Bayesian Beta model — the mathematically optimal estimate of true usefulness:
```
recall({
  query: "auth patterns",
  feedback: [
    { id: "abc123", useful: true },   // Beta(2,1) → ×1.33 next time
    { id: "def456", useful: false }   // Beta(1,2) → ×0.67 next time
  ]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Goldentrii/AgentRecall-MCP](https://github.com/Goldentrii/AgentRecall-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
