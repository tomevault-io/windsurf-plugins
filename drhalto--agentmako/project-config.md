---
trigger: always_on
description: This file is meant to be copied into your target project's `AGENTS.md`,
---

# Mako MCP Usage (paste into AGENTS.md or CLAUDE.md)

This file is meant to be copied into your target project's `AGENTS.md`,
`CLAUDE.md`, or equivalent agent-instructions file. It teaches a coding
agent how to use Mako's MCP tools effectively before reading or editing
your code.

Mako is registered with MCP clients as `mako-ai`:

```json
{
  "mako-ai": {
    "command": "agentmako",
    "args": ["mcp"]
  }
}
```

In Claude Code, Mako tools usually appear as `mcp__mako-ai__<toolName>`.
The examples below use the bare tool name for readability.

## Operating Model

Mako is a deterministic project context engine, not a replacement for
normal coding discipline. Use it to narrow the work: relevant files,
symbols, routes, schema objects, findings, freshness, and risks. Then
use normal reads, edits, tests, and shell commands to implement and
verify.

Prefer Mako before broad grep/file walking when the question is about
project structure, cross-file impact, database usage, routing, auth, or
known findings. Prefer `live_text_search` or shell `rg` when you need
exact current disk text after edits.

Mako has two evidence modes:

- Indexed/Reef evidence: fast and structured, but tied to the last index
  or persisted fact snapshot.
- Live evidence: current filesystem or live database. Use this when
  line numbers, edited files, or recently created files matter.

Do not treat answer stability as freshness. A stable indexed answer can
still be stale relative to disk. Check `project_index_status`,
per-evidence freshness fields, or `live_text_search` before relying on
exact lines after edits.

## First Tool To Use

For a vague task, start with `context_packet`.

```json
{
  "request": "debug why manager onboarding role checks are failing",
  "includeInstructions": true,
  "includeRisks": true,
  "includeLiveHints": true,
  "freshnessPolicy": "prefer_fresh",
  "budgetTokens": 4000
}
```

Read the returned `primaryContext`, `relatedContext`, `activeFindings`,
`risks`, `scopedInstructions`, `recommendedHarnessPattern`, and
`expandableTools`. Then follow the normal harness loop: read the primary
files, search references, edit surgically, and verify.

When the task already names files, include them:

```json
{
  "request": "review auth impact of this change",
  "focusFiles": ["lib/auth/dal.ts", "app/dashboard/manager/layout.tsx"],
  "includeInstructions": true,
  "includeRisks": true
}
```

## Fast Follow-Up Batches

Use `tool_batch` for independent read-only lookups. It reduces MCP
round trips and keeps results labeled.

```json
{
  "verbosity": "compact",
  "continueOnError": true,
  "ops": [
    {
      "label": "freshness",
      "tool": "project_index_status",
      "args": { "includeUnindexed": false }
    },
    {
      "label": "auth-conventions",
      "tool": "project_conventions",
      "args": { "limit": 20 }
    },
    {
      "label": "open-loops",
      "tool": "project_open_loops",
      "args": { "limit": 20 }
    }
  ]
}
```

`tool_batch` is read-only. It rejects mutation tools such as
`project_index_refresh`, `working_tree_overlay`, `diagnostic_refresh`,
`db_reef_refresh`, `finding_ack`, and `finding_ack_batch`.

Use `verbosity: "compact"` or per-op `resultMode: "summary"` when
querying noisy tools like `cross_search`, `recall_tool_runs`, or
project-wide Reef views.

## Freshness And Indexing

Use `project_index_status` before trusting indexed line numbers or after
large edits.

```json
{
  "includeUnindexed": false
}
```

Use `includeUnindexed: true` only when you need to discover new files on
disk; it costs a filesystem walk.

If Mako reports stale, dirty, unknown, or missing indexed evidence, use
one of these:

- `live_text_search` for exact current text without reindexing.
- `project_index_refresh` with `mode: "if_stale"` when the index should
  be refreshed.
- `project_index_refresh` with `mode: "force"` only when the indexed
  AST/search results appear wrong.
- `working_tree_overlay` to snapshot working-tree file facts without
  reparsing AST/imports/routes/schema.

Example:

```json
{
  "mode": "if_stale",
  "reason": "Need fresh indexed context before editing auth route"
}
```

## Search And Code Intelligence

Use `cross_search` for broad indexed search across code chunks, routes,
schema objects, RPC/trigger bodies, and memories.

```json
{
  "term": "admin_audit_log",
  "limit": 20
}
```

Use `live_text_search` for exact current text on disk. It defaults to
fixed-string search.

```json
{
  "query": "verifySession(",
  "pathGlob": "lib/**/*.ts",
  "fixedStrings": true,
  "maxMatches": 100
}
```

Use `ast_find_pattern` for structural TS/JS/TSX/JSX matches.

```json
{
  "pattern": "supabase.from($TABLE)",
  "languages": ["ts", "tsx"],
  "pathGlob": "app/**/*.tsx",
  "maxMatches": 200
}
```

Use these focused code tools when the shape is known:

- `repo_map`: token-budgeted project outline.
- `symbols_of`, `exports_of`: symbol and export surfaces for a file.
- `imports_deps`, `imports_impact`, `imports_hotspots`,
  `imports_cycles`: import graph questions.
- `graph_neighbors`, `graph_path`, `flow_map`: graph traversal and flow
  context.
- `trace_file`: explain one file.
- `route_trace`, `route_context`: route resolution and route
  neighborhood.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drhalto/agentmako](https://github.com/drhalto/agentmako) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
