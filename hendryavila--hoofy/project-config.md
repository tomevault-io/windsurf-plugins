---
trigger: always_on
description: Hoofy is an **AI development companion** — an MCP server written in Go 1.25 that provides persistent memory, adaptive change management, and spec-driven development. It prevents AI hallucinations by forcing structured requirements before coding, remembers context across sessions, and adapts its workflow to the size and type of each change.
---

# AGENTS.md — Hoofy Development Guide

## Project Overview

Hoofy is an **AI development companion** — an MCP server written in Go 1.25 that provides persistent memory, adaptive change management, and spec-driven development. It prevents AI hallucinations by forcing structured requirements before coding, remembers context across sessions, and adapts its workflow to the size and type of each change.

**Binary**: `hoofy` (CLI with `serve`, `update`, `version` commands)
**Transport**: stdio (MCP protocol)
**Dependency**: `mcp-go v0.44.0` (Mark3Labs MCP SDK)

## Architecture

### Directory Structure

```
cmd/hoofy/              Entry point — CLI argument parsing, server startup, graceful shutdown
internal/
├── changes/            Change pipeline — types, flows, store, state machine
├── config/             Project config persistence (hoofy.json) — types, Store interface, FileStore
├── memory/             Persistent memory — SQLite store, FTS5 search, sessions, observations
├── memtools/           MCP memory tool handlers — 19 tools for save, search, context, sessions, relations, progress
├── pipeline/           Pipeline state machine — stage transitions, Clarity Gate thresholds
├── prompts/            MCP prompts — /sdd-start, /sdd-status, /sdd-stage-guide, /sdd-memory-guide, /sdd-change-guide, /sdd-bootstrap-guide
├── resources/          MCP resources — project status resource
├── server/             Composition root — wires all dependencies, registers tools/prompts/resources
├── templates/          Go templates for stage artifacts (guided + expert mode variants)
├── tools/              MCP tool handlers — one file per tool (init, principles, charter, specify, clarify, design, tasks, validate, context, change, adr, audit, bridge, suggest_context, review)
└── updater/            Self-update system — GitHub releases API, binary replacement
```

### Design Principles

- **SRP**: One file = one tool. Config handles persistence, pipeline handles business rules.
- **DIP**: Tools depend on interfaces (`config.Store`, template renderer), not concretions. `server.go` is the composition root that wires everything.
- **OCP**: New tools/stages are added without modifying existing ones.
- **Liskov**: Both modes (guided/expert) use the same pipeline interface, differing only in thresholds and templates.

### Key Abstractions

```
config.Store (interface)
├── config.Loader — reads hoofy.json
└── config.Saver — writes hoofy.json
    └── config.FileStore (concrete) — filesystem implementation

memory.Store (interface)
├── SaveObservation, SearchObservations, GetContext, GetTimeline
├── Session management (Start, End, Summary)
└── memory.SQLiteStore (concrete) — SQLite + FTS5 implementation
```

Tools receive `config.Store`, `memory.Store`, and `templates.Renderer` via constructor injection in `server.go`.

### Pipeline State Machine

```
init → principles → charter → specify → business-rules → clarify (Clarity Gate) → design → tasks → validate
```

- 9 stages, sequential — cannot skip ahead.
- The `principles` stage captures golden invariants and coding standards.
- The `charter` stage replaces the old `propose` stage with expanded enterprise-grade fields.
- The Clarity Gate at `clarify` blocks advancement until clarity score meets threshold:
  - Guided mode: 70/100
  - Expert mode: 50/100
- Stage status: `pending` → `in_progress` → `completed`
- State persisted in `docs/hoofy.json` in the user's project directory.

### Change Pipeline State Machine

```
create → context-check → [describe/charter/scope] → [spec] → [clarify] → [design] → tasks → verify
                          └─── stages selected automatically based on type × size ───┘
```

- 4 types (feature, fix, refactor, enhancement) × 3 sizes (small, medium, large) = 12 flow variants
- One active change at a time
- Artifacts stored in `docs/changes/<slug>/`
- Completed changes archived to `docs/history/<slug>/`

### MCP Components

| Type | Components |
|------|-----------|
| **Tools (Project)** | `sdd_init_project`, `sdd_create_principles`, `sdd_create_charter`, `sdd_generate_requirements`, `sdd_create_business_rules`, `sdd_clarify`, `sdd_create_design`, `sdd_create_tasks`, `sdd_validate`, `sdd_get_context`, `sdd_reverse_engineer`, `sdd_bootstrap` |
| **Tools (Change)** | `sdd_change`, `sdd_context_check`, `sdd_change_advance`, `sdd_change_status`, `sdd_adr` |
| **Tools (Standalone)** | `sdd_explore`, `sdd_suggest_context`, `sdd_review`, `sdd_audit` |
| **Tools (Memory)** | `mem_save`, `mem_save_prompt`, `mem_search`, `mem_context`, `mem_timeline`, `mem_get_observation`, `mem_relate`, `mem_unrelate`, `mem_build_context`, `mem_session_start`, `mem_session_end`, `mem_session_summary`, `mem_stats`, `mem_capture_passive`, `mem_delete`, `mem_update`, `mem_suggest_topic_key`, `mem_progress`, `mem_compact` |
| **Prompts** | `/sdd-start`, `/sdd-status`, `/sdd-stage-guide`, `/sdd-memory-guide`, `/sdd-change-guide`, `/sdd-bootstrap-guide` |
| **Resources** | Project status resource |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HendryAvila/Hoofy](https://github.com/HendryAvila/Hoofy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
