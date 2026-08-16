---
trigger: always_on
description: Instructions for AI coding agents working on this repo. Keep changes surgical: touch only what the task requires, match existing style, preserve behavior in refactors.
---

# marm-memory - Agent Instructions

Instructions for AI coding agents working on this repo. Keep changes surgical: touch only what the task requires, match existing style, preserve behavior in refactors.

## Architecture

MARM is a local-first MCP memory server: Python FastAPI in `marm-mcp-server/`, package `marm_mcp_server/`.

- **14 public MCP tools**: 7 core memory, 5 code graph, 2 concept graph. HTTP and STDIO must stay in exact parity.
- **HTTP transport**: `marm_mcp_server/server.py`; tools are whitelisted in `MCP_TOOL_OPERATIONS`. A tool not in that list does not exist over HTTP.
- **STDIO transport**: `marm_mcp_server/server_stdio.py` owns the `FastMCP` app and seven core `@mcp.tool()` wrappers. Graph/concept bodies live in `services/stdio_graph_tools.py` and are explicitly registered after the core tools so `tools/list` order stays stable. Never fork behavior between transports.
- **Endpoint logic** lives in `marm_mcp_server/endpoints/` split by surface (memory, logging, notebook, session, compaction, graph, concepts, system). Shared helpers stay in `core/`.
- **Storage**: SQLite WAL at `~/.marm/marm_memory.db` (connection pool, FTS5 external-content index `memories_fts`, `memory_chunks` for long-memory chunking). The concept graph uses its own database `~/.marm/index/marm_index.db` with its own pool. Never share connections between the two.
- **Write path**: all memory writes go through the serialized async write queue (one worker). Do not add write paths that bypass it. `marm_log_entry` dual-writes: a `log_entries` row plus a semantic memory in `memories` (via the queue); a semantic-store failure must never fail the log write.
- **Code graph**: a pinned external binary (codebase-memory-mcp) supervised as a child process over newline-delimited JSON-RPC (`core/graph_supervisor.py`, `core/graph_client.py`). It starts lazily and runs degraded on failure. Graph or concept failures must never break the 7 core memory tools.
- **Both graphs index themselves**, on by default, one background worker each on both transports. Concept extraction is queue-driven: a write enqueues a durable outbox row in the same transaction as the memory (`core/concept_worker.py`). Code indexing is poll-driven: a git signature per indexed repo, re-indexing on a commit and every cycle while the tree is dirty (`core/graph_index_worker.py`). Neither may be made to block a write, a recall, or startup.
- **Cross-process serialization is a leased DB row, never an asyncio lock.** `core/lease_lock.py` owns the mechanics; `concept_build_lock` and `graph_index_lock` are its two bindings, deliberately separate rows. HTTP and STDIO are separate processes, so an in-process lock protects nothing. Every code-index call AND `delete_project` take the graph gate. Release is driven by the engine call's completion, not the awaiting task: `asyncio.to_thread` cancellation cancels the await and leaves the thread writing.
- **Runtime switches live in the DB, not just the environment** (`core/runtime_flags.py`). A saved override beats the env var so a Dockerfile cannot silently re-enable what a user turned off, and both workers re-read per cycle so no restart is needed. Any new background worker follows this: read the flag every cycle, and start the loop even when off so it can be turned on from another process.
- **Graph-aware recall**: `marm_smart_recall` keeps primary memory ranking authoritative and may add bounded `graph_context` from the isolated concept database. Graph enrichment is read-only and fail-open; trim graph details before primary results when enforcing response limits.
- **Embeddings**: one fastembed `jinaai/jina-embeddings-v2-small-en` encoder (512 dimensions), lazy-loaded and serialized behind a lock. Writes must succeed even when the encoder is unavailable. Existing data requires `marm-mcp-server --migrate-embeddings` before restart when upgrading from MiniLM.

## Consistency Rules

**When adding or removing an MCP tool, update ALL of the following:**

1. `marm_mcp_server/endpoints/<surface>.py` - the implementation
2. `marm_mcp_server/server.py` - route + `MCP_TOOL_OPERATIONS` whitelist
3. `marm_mcp_server/server_stdio.py` - STDIO bootstrap/registration and matching wrapper or service path
4. `marm-mcp-server/server.json` - tools array
5. `scripts/find-tools.py` - `CANONICAL_TOOLS` list
6. Docs with full tool lists: `README.md`, `docs/PROTOCOL.md`, `docs/PROTOCOL-LITE.md`, and their `marm-mcp-server/marm_mcp_server/resources/marm-docs/` copies, plus tool counts in FAQ
7. Tests covering both transports

Then run `python scripts/find-tools.py`; every surface must report OK.

**README variants:**

- Root `README.md` is the single source of truth.
- `marm-mcp-server/README.md` is the PyPI variant (adds the `mcp-name:` header and two image divs) and is maintained separately.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lyellr88/MARM-Systems](https://github.com/Lyellr88/MARM-Systems) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
