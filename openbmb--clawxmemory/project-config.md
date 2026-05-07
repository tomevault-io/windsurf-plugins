---
trigger: always_on
description: This file is the repository-wide guide for coding agents working on ClawXMemory.
---

# AGENTS.md

This file is the repository-wide guide for coding agents working on ClawXMemory.

Use this file for agent-facing implementation guidance. Keep user-facing product and usage documentation in [README.md](README.md) and [docs/README_zh.md](docs/README_zh.md).

## Repository Scope

ClawXMemory is an OpenClaw `memory` plugin. The repository contains:

- the plugin runtime and hook wiring
- the file-based long-term memory store
- background indexing and Dream organization
- answer-time recall and prompt injection
- the local dashboard and trace views
- prompt assets and agent workflows

The current architecture is **markdown-first**:

- long-term memory lives in markdown files under the memory directory
- SQLite stores runtime control-plane state only
- recall, Dream, and UI all operate on file-memory, not on legacy aggregated tables

## Source Map

- `clawxmemory/src/index.ts`
  Plugin entry, service registration, tool registration, and prompt section registration.
- `clawxmemory/src/runtime.ts`
  Runtime composition, queue/timer orchestration, retrieval entrypoints, UI state, and memory boundary diagnostics.
- `clawxmemory/src/hooks.ts`
  Hook wiring for `before_prompt_build`, `before_message_write`, `agent_end`, `before_reset`, and internal message/command filtering.
- `clawxmemory/src/tools.ts`
  User-facing tool contracts: `memory_search`, `memory_overview`, `memory_list`, `memory_get`, and `memory_flush`.
- `clawxmemory/src/core/pipeline/heartbeat.ts`
  Background indexing pipeline from raw sessions into file-memory candidates and user profile rewrites.
- `clawxmemory/src/core/retrieval/reasoning-loop.ts`
  Single-project recall flow, project shortlist building, header-scan manifest selection, and final context assembly.
- `clawxmemory/src/core/review/dream-review.ts`
  Dream planning, project rewrite execution, file deletion, and user-profile rewrite orchestration.
- `clawxmemory/src/core/skills/llm-extraction.ts`
  Structured LLM prompts and parsers for extraction, route selection, project selection, manifest selection, Dream planning, and profile rewrites.
- `clawxmemory/src/core/storage/sqlite.ts`
  SQLite runtime state, raw session queue, indexing settings, and recent trace persistence.
- `clawxmemory/src/core/file-memory.ts`
  Markdown-backed memory store for `user-profile.md`, `project.meta.md`, `Project/*.md`, `Feedback/*.md`, and derived `MEMORY.md`.
- `clawxmemory/src/ui-server.ts`
  Local dashboard server.
- `clawxmemory/ui-source/*`
  Dashboard frontend.

## Architecture Contract

ClawXMemory is intentionally `plugin-first`, not `skills-only`.

Use the plugin runtime for:

- lifecycle hooks
- automatic raw session capture
- background indexing
- retrieval-time prompt injection
- Dream execution
- tool registration
- the local UI server

Use skills and prompt assets for:

- extraction rules
- recall routing and project selection
- Dream planning and project rewrite instructions
- retrieval context rendering
- agent workflows built on top of the plugin tools

Do not redesign the system as a pure skill layer unless the OpenClaw lifecycle constraints also change.

## Memory Model

The current system has two distinct layers.

### 1. Runtime state in SQLite

SQLite is a control-plane store, not the long-term memory truth source.

It persists:

- `l0_sessions`
  Raw captured conversation sessions waiting for or already processed by indexing.
- `pipeline_state`
  Runtime settings, recent case/index/dream traces, timestamps, and other plugin state.

Field semantics that should stay stable unless a migration is introduced:

- `indexed`
  Only on `L0SessionRecord`. Marks whether the raw session has already been consumed by the indexing pipeline.
- `source`
  Only on `L0SessionRecord`. Tracks where raw sessions came from, such as `openclaw`, `skill`, or `import`.
- `createdAt`
  First persistence time for a record.
- `updatedAt`
  Last runtime rewrite time for mutable state rows.

### 2. Long-term memory on disk

Long-term memory is stored as markdown files:

- `global/User/user-profile.md`
- `projects/<projectId>/project.meta.md`
- `projects/<projectId>/Project/*.md`
- `projects/<projectId>/Feedback/*.md`
- `global/MEMORY.md` and `projects/*/MEMORY.md`

`MEMORY.md` is a derived directory file for UI and debugging. It is not the recall truth source.

## Runtime Flow

### Answer-time Recall

The main answer-time hook is `before_prompt_build`.

Current flow:

1. Read the current user query and recent messages.
2. Call `ReasoningRetriever.retrieve(...)`.
3. Run the memory gate with route values:
   - `none`
   - `user`
   - `project_memory`
4. Load the global user profile when memory is needed.
5. For `project_memory`, build a formal project shortlist and let the model select at most one project.
6. Scan file headers from the selected project's `Project/*.md` and `Feedback/*.md`.
7. Let the model choose top-k memory files from the compact manifest.
8. Load selected files with full-text preference and hard limits.
9. Inject the rendered recall context through `prependSystemContext`.

Important:

- recall is **single-project by design**
- assistant replies are not used as project-resolution evidence
- `project.meta.md` is fixed context once a project is selected

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenBMB/ClawXMemory](https://github.com/OpenBMB/ClawXMemory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
