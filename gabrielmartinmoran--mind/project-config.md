---
trigger: always_on
description: This document describes the **mind** project: its architecture, behavior, technical choices, and how to use it. It is intended for AI agents and human maintainers. **Agents that modify this codebase must keep this file updated** when they change architecture, add commands, change config, or alter behavior (see [Keeping this document updated](#keeping-this-document-updated)).
---

# AGENTS.md — Project guide for AI agents and maintainers

This document describes the **mind** project: its architecture, behavior, technical choices, and how to use it. It is intended for AI agents and human maintainers. **Agents that modify this codebase must keep this file updated** when they change architecture, add commands, change config, or alter behavior (see [Keeping this document updated](#keeping-this-document-updated)).

---

## 1. Project overview

**mind** is a CLI tool for persistent long-term memory — tracking thoughts, ideas, tasks, and knowledge. Data is organized into named **spaces**, each containing **memories** with full-text search, tags, links, and a 3-tier CPU-cache-style access-frequency system.

- **Runtime:** [Bun](https://bun.sh/)
- **Language:** TypeScript (strict mode, ESNext)
- **Entry point:** the **`mind`** Bash script at project root; invokes `src/mind.ts`. Supports subcommands: `serve` (HTTP server), `mcp` (MCP server), `setup` (agent configuration), and `update` (self-update from GitHub releases). Also supports `--complete` flag to delegate to `src/complete.ts` (not yet implemented).
- **Persistence:** SQLite database at `data/mind.db` (path configurable via `MIND_DATA_DIR` env var or `MIND_DB_PATH` for full path override). The legacy `brain.json` is supported as a migration source via `mind import`.
- **RAG/Embeddings:** Optional semantic search via OpenAI `text-embedding-3-small`. Enable with `MIND_RAG=true` + `OPENAI_API_KEY`. Embeddings stored as BLOBs in SQLite; generated fire-and-forget on add/update.
- **Layout:** **`src/`** contains CLI, MCP, and API server code. **`test/`** contains backend/CLI tests. **`web/`** contains frontend source (`web/src`), styles (`web/styles`), static assets (`web/assets`), static HTML shell (`web/public`), and web-specific tests (`web/test`). **`scripts/`** contains E2E test scripts.
- **Neural Map MVP:** web SPA includes a read-only per-space graph view using a minimal graph API payload and on-demand memory detail fetch, with deterministic best-effort anti-overlap placement, 25-char visible label truncation (full name retained via accessibility/tooltip), and higher zoom ceiling for dense maps.
- **Web SPA URL routing:** client-side URL contract is `/'` (home) and `/spaces/{encodedSpace}?view=list|map&memory={encodedMemory?}` with reload restore, browser back/forward support (`popstate`), and safe fallback canonicalization when route params are invalid.

---

## 2. Architecture

### 2.1 High-level flow

```
User → ./mind <command> [args] [--flag value]
         ↓
    mind (Bash script at repo root)
         ↓
    bun run src/mind.ts "$@"
         ↓
    executeCommand(args, store, logger)
         ↓
    CLI command registry (atomic command modules) → command-executor (dispatch)
         ↓
    MindStore (SQLite) + Logger (stdout/stderr)
```

- **Entry:** `src/mind.ts` creates store/logger and delegates all command handling to `executeCommand` from `src/cli/command-executor.ts`.
- **Commands:** Declared as atomic modules in `src/cli/commands/*.ts` and registered by `src/cli/commands/index.ts`. `src/cli/command-executor.ts` acts as dispatcher/registry.
- **Storage:** All persistent data goes through the `MindStore` interface (defined in `src/store/mind-store.ts`), implemented by `createSqliteStore` (`src/store/sqlite-store.ts`). Uses bun's native `bun:sqlite`.
- **FTS:** Full-text search uses SQLite's FTS5 with a porter tokenizer. FTS is synced **manually** (bun:sqlite has a bug with content-sync triggers — see [§ 3](#3-technical-considerations)).
- **Output:** All user-facing messages go through the `Logger` interface (`src/helpers/logger.ts`), so tests can swap in a mock logger.
- **Web/API:** The single canonical HTTP server is `src/api/server.ts` (via `mind serve`), and it serves API routes plus static web files from the `web/` tree.

### 2.2 Main modules and responsibilities

| Module                | Path                                                                 | Responsibility                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GabrielMartinMoran/mind](https://github.com/GabrielMartinMoran/mind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
