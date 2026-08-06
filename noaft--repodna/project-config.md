---
trigger: always_on
description: This file guides coding agents working in the RepoDNA repository.
---

# AGENTS.md

This file guides coding agents working in the RepoDNA repository.

## Mission

RepoDNA is building the memory layer for code tools.

The product goal is not "another assistant." The goal is persistent repository memory:

- ingest repository history and structure
- store durable knowledge in a local graph
- expose that knowledge to tools through local APIs and MCP
- reduce repeated rediscovery across sessions

When making changes, optimize for continuity, retrieval quality, and operational simplicity.

## What Matters Most

Agents should preserve these project priorities:

1. Durable repository memory beats short-lived prompt memory.
2. Local-first workflows are preferred over cloud-dependent assumptions.
3. MCP ergonomics matter because RepoDNA is a substrate for other tools.
4. Storage path behavior must stay predictable across CLI, API, and MCP flows.
5. Changes should improve context recovery, not just raw feature count.

## Codebase Map

- [src/main.rs](/abs/path/d:/Git/RepoDNA/src/main.rs:1)
  CLI entry point. Wires commands such as `setup`, `build`, `update`, `viewer`, and `serve`.
- [src/ingestion/mod.rs](/abs/path/d:/Git/RepoDNA/src/ingestion/mod.rs:1)
  Core graph-building logic. Commits, authors, files, directories, functions, edges, hotspots, ownership, and rebuild/update flow live here.
- [src/api/mod.rs](/abs/path/d:/Git/RepoDNA/src/api/mod.rs:1)
  Graph API server over the persisted database.
- [src/bin/repodna_mcp.rs](/abs/path/d:/Git/RepoDNA/src/bin/repodna_mcp.rs:1)
  Global MCP server. It exposes graph-backed node search and durable node-context tools, and can auto-resolve the active git workspace.
- [src/repo_registry.rs](/abs/path/d:/Git/RepoDNA/src/repo_registry.rs:1)
  Local registry of repositories configured through `RepoDNA setup`; used by global MCP fallback behavior.
- [src/settings.rs](/abs/path/d:/Git/RepoDNA/src/settings.rs:1)
  Central place for RepoDNA environment-driven settings.
- [src/repodna_paths.rs](/abs/path/d:/Git/RepoDNA/src/repodna_paths.rs:1)
  Resolves storage paths for `graph.db` and `state.json`.
- [README.md](/abs/path/d:/Git/RepoDNA/README.md:1)
  Product framing and operator-facing usage.
- [docs/VISION.md](/abs/path/d:/Git/RepoDNA/docs/VISION.md:1)
  Strategic direction. Keep new work aligned with the "memory layer" story.

## Trusted Commands

Use these first when validating changes:

```powershell
cargo check
```

Build the graph for the repo:

```powershell
RepoDNA setup D:\Git\RepoDNA
```

Use a fixed shared DB path:

```powershell
$env:REPODNA_DB_PATH='D:\RepoDNA\.repodna\graph.db'
RepoDNA setup D:\Git\RepoDNA
```

Run the MCP server:

```powershell
cargo run --bin repodna_mcp
```

Serve the graph API:

```powershell
cargo run -- serve D:\Git\RepoDNA 127.0.0.1:3000
```

## Storage Rules

Storage behavior is a core part of the product. Be careful when changing it.

- `REPODNA_DB_PATH` pins RepoDNA to one concrete SQLite file.
- With no storage env, RepoDNA stores graph/state files inside the target repo at `.repodna/`.
- `REPODNA_HOME` sets a shared storage root for RepoDNA-managed per-repo graph/state files.
- `RepoDNA setup <repo>` also registers the repo in RepoDNA's local registry so the global MCP server can resolve known repos.
- `Settings::from_env()` in [src/settings.rs](/abs/path/d:/Git/RepoDNA/src/settings.rs:1) is the canonical place for env-driven settings.
- Path resolution belongs in [src/repodna_paths.rs](/abs/path/d:/Git/RepoDNA/src/repodna_paths.rs:1), not scattered through the codebase.

If you add a new environment variable:

1. Define it in `src/settings.rs`.
2. Thread it through the relevant path or runtime logic.
3. Update `.env.example` if user-configurable.
4. Update `README.md` if it affects setup or operator behavior.

## MCP Rules

The MCP server must be easy to launch and resilient in local environments.

- Avoid unnecessary startup dependencies before the MCP handshake.
- If a setting already gives a direct DB path, prefer using it instead of rediscovering the repo.
- Prefer one global MCP server named `repodna`; do not require users to register one MCP server per repo.
- With no repo argument, MCP should discover the current git workspace and resolve that repo's graph DB.
- If discovery fails, MCP may use the local repo registry only when that fallback is unambiguous.
- Remember that MCP failures often look like handshake failures even when the real problem is early process exit.
- Keep MCP tool outputs schema-safe. Root output schemas must be objects, not arrays.

## RepoDNA Memory-First Workflow

When an agent needs to understand repository code, prefer RepoDNA memory before broad filesystem search:

1. Before reading files broadly, rebuilding repository context, running wide text search, or opening many files, ask RepoDNA first.
2. On a new or unfamiliar repository, call `first_look`. If it returns `bootstrap_needed`, read its recommended nodes and then call `add_node_context` for each node you genuinely understand.
3. When returning after code changes, call `context_health`. If it reports stale nodes, treat saved summaries as orientation only, inspect the current source or diff, then call `update_node_description` with the exact `node_id`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noaft/RepoDNA](https://github.com/noaft/RepoDNA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
