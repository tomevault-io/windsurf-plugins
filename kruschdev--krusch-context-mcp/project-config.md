---
trigger: always_on
description: > Unified MCP server: PG-Git codebase search + Homelab episodic memory + Holographic Nuggets + External docs search.
---

# krusch-context-mcp — Agent Context

> Unified MCP server: PG-Git codebase search + Homelab episodic memory + Holographic Nuggets + External docs search.

> **Last audit**: 2026-07-17 | **Version**: 1.2.0 | **Tools**: 32

## Architecture Overview

This project is a single MCP server (stdio transport) that unifies three subsystems into one process:

1. **Episodic Memory** — Vector-embedded memories (`bugs`, `lessons`, `priorities`, `outcomes`, `activity`) stored in hybrid SQLite + Postgres. These maintain agent state across sessions, track workarounds, and document learnings. For a comprehensive guide, see [EPISODIC_MEMORY.md](file:///home/krusch/homelab/projects/krusch-context-mcp/docs/EPISODIC_MEMORY.md).
2. **Holographic Nuggets** — Lightweight key-value steering facts (preferences, conventions, corrections) with semantic retrieval
3. **Codebase Search** — Semantic search over all indexed source files via the sibling `pg-git` project
4. **Proactive Auditor (Memory Agent Loop)** — Background trajectory auditor that verifies actions against past rules and updates weights based on developer feedback loops (Direct-OPD/PUST)

All four share a single `pg.Pool` connection to `kruschdb` and a single fleet-balanced Ollama embedding pipeline.

### Lakebase Architecture (Compute/Storage Decoupling)

Project-scoped data follows a two-tier model:
- **Compute Cache**: Per-project SQLite databases at `<project>/.agent/memory.db` — zero-latency reads
- **Object Storage**: Durable Postgres tables (`ide_agent_memory`, `ide_agent_nuggets`, `interaction_memory`) — fleet-wide persistence
- **Sync**: Async write-behind (SQLite → Postgres) on every write; read-ahead pull (Postgres → SQLite) on first project access

### Storage Routing Rules

| Operation | `project`/`active_project` provided | `project`/`active_project` omitted |
|-----------|--------------------------------------|-------------------------------------|
| **Write memory** | SQLite + async PG push | Postgres directly |
| **Search memory** | Merge: SQLite + Postgres (SQLite gets +0.3 bias) | Postgres only |
| **Write nugget** (`kind: 'project'`) | SQLite + async PG push | Postgres fallback |
| **Write nugget** (`kind: 'user'`/`'agent'`) | Always Postgres | Always Postgres |
| **Delete/Update memory** | SQLite (via `source_project`) | Postgres |

## Key Constraints

- **Database schema (`ide_agent_memory`)**: Must maintain `project` and `tags` columns added via dynamic migration. Do NOT alter the column order or types — backward compatibility with existing episodic records is critical.
- **pg-git dependency**: All DB pooling and embedding logic is imported from the sibling `pg-git` project via `file:` link in `package.json`. This project has NO `.env` of its own — it inherits `pg-git/.env` configuration.
- **Ollama models**: Embeddings use `bge-large` (1024 dims). Tag generation uses `llama3.2` for keyword extraction. Both are dispatched through a shared LLM queue at `../../../lib/llm-queue.js`.
- **Memory categories**: Valid values for episodic memory: `priorities`, `bugs`, `outcomes`, `lessons`, `activity`. Valid values for Company Brain v2 substrate / feedback memory: `alignment_signal`.
- **Strict Project Isolation**: For `search_code` and `deep_search`, if the `project` argument is provided, it must EXACTLY match a known repository name. The server will throw an error rather than falling back to a global codebase search.
- **Nugget kinds**: Only 3 valid values: `project`, `user`, `agent`.
- **Temporal decay**: Search results are weighted by `similarity × e^(-0.01 × age_in_days)`. A memory's relevance drops ~26% after 30 days of inactivity.

## Source Files

| File | Responsibility |
|------|---------------|
| `src/index.js` | MCP server entry point — tool registration, routing, DB migration, codebase/docs/health tools |
| `src/memory-engine.js` | Episodic memory CRUD (add, search, list, delete, update, consolidate, compile_state) |
| `src/v2-engine.js` | Company Brain v2 substrate (write_state, resolve_conflict, provenance, ontology, lens, graph, link_blob) |
| `src/nuggets-engine.js` | Holographic Nuggets CRUD (remember, nudges, forget, list) with hybrid SQLite/Postgres routing |
| `src/sqlite-engine.js` | Lakebase SQLite layer — project DB init, pull/push sync, cosine similarity helper |
| `src/llm-tags.js` | Shared LLM tag generation helper (used by memory-engine + v2-engine) |
| `src/think-engine.js` | cited context synthesis, conflict detection, and gap analysis |
| `src/skills-engine.js` | Agent skills loader and prompt registry engine |
| `src/proactive-engine.js` | Trajectory auditor (proactive_nudge) and feedback collector (nudge_feedback) |

## Tool Surface (32 tools)

| Tool | Source | Key Parameters |
|------|--------|---------------|
| `krusch_context_add_memory` | `memory-engine.js` | `category`★, `content`★, `project`, `tags` |
| `krusch_context_search_memory` | `memory-engine.js` | `category`★, `query`★, `limit`, `active_project` |
| `krusch_context_list_memories` | `memory-engine.js` | `category`★, `project`, `limit` |
| `krusch_context_delete_memory` | `memory-engine.js` | `id`★, `source_project` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kruschdev/krusch-context-mcp](https://github.com/kruschdev/krusch-context-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
