---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`ccpulse` is a local-first dashboard for Claude Code session analytics. A daemon watches `~/.claude/projects/**/*.jsonl`, indexes events into SQLite, and serves a REST + SSE API consumed by an embedded React SPA. Single npm package, pnpm workspaces internally.

## Common commands

Run from the repo root unless noted.

```bash
# install (first time, or after dependency changes)
pnpm install

# native binding for better-sqlite3 — required after a Node version change
pnpm rebuild better-sqlite3

# package-level builds (each emits dist/)
pnpm --filter ccpulse-core build
pnpm --filter ccpulse-daemon build
pnpm --filter ccpulse build              # the CLI package, name is bare `ccpulse`
pnpm --filter ccpulse-web build

# embed the web bundle into the daemon (must run after web build)
pnpm exec tsx scripts/embed-web.ts

# typecheck everything
pnpm -r typecheck

# tests (vitest, currently only in core)
pnpm --filter ccpulse-core test
pnpm --filter ccpulse-core test -- src/parser.test.ts -t "ai-title"   # single test

# dev loop — daemon + Vite with API proxy
pnpm dev:daemon      # terminal 1 — Hono server on :7878, no UI bundle
pnpm dev:web         # terminal 2 — Vite on :5174, proxies /api → :7878
```

The full release build is: `core` → `web` → `daemon` → `cli` → `embed-web.ts`. `package.json`'s `build` script runs them in order.

**The CLI bundles `ccpulse-core` and `ccpulse-daemon` into `packages/cli/dist/bin.js`** (tsup with only `chokidar` external). The globally-linked `ccpulse` binary (`~/Library/pnpm/ccpulse` → this repo's `packages/cli/dist/bin.js`) therefore runs whatever core code was bundled at the time `pnpm --filter ccpulse build` last ran. After editing anything in `core/` or `daemon/`, rebuild the CLI too — restarting the daemon alone is not enough. `pnpm dev:daemon` uses tsx on source and does not have this issue.

**`pnpm --filter ccpulse build` clears `packages/cli/dist/` (tsup `--clean`), which deletes `cli/dist/embedded` too.** `embed-web.ts` copies the web bundle to both `packages/daemon/embedded` and `packages/cli/dist/embedded`. Always re-run `pnpm exec tsx scripts/embed-web.ts` after a CLI rebuild, or the daemon serves the "UI not built" page. Top-level `pnpm build` does this in the right order.

## Running the built CLI

```bash
node packages/cli/dist/bin.js daemon          # foreground server
node packages/cli/dist/bin.js open            # opens browser scoped to $PWD
node packages/cli/dist/bin.js status
node packages/cli/dist/bin.js reindex         # drops SQLite, rebuilds on next daemon start
```

Env: `CCPULSE_PORT` (default 7878), `CCPULSE_DB` (default `~/.ccpulse/ccpulse.db`), `CCPULSE_CLAUDE_DIR` (override the JSONL root for tests).

## Data flow you need to understand

```
~/.claude/projects/<encoded-cwd>/<sessionId>.jsonl  ← canonical event log (append-only)
        │
        ▼
  watcher.ts (chokidar)  ─►  indexer.ts  ─►  SQLite (~/.ccpulse/ccpulse.db)
        │                                          │
        ▼                                          ▼
  EventEmitter                               queries.ts
        │                                          │
        └────► server.ts (Hono) ──► /api/* + /api/stream (SSE) ──► web SPA
```

- JSONL is the source of truth. SQLite is a rebuildable index. Event UUIDs are unique; ingest is idempotent (`INSERT … ON CONFLICT(uuid) DO NOTHING`).
- The watcher tracks per-file byte offsets in `file_offsets`, so a daemon restart only re-ingests new bytes. If the file rotates (inode changes or size shrinks) the offset resets to 0.
- SSE pushes `ingest` events from the watcher's EventEmitter; the web client uses them only as cache-invalidation triggers, then refetches. The client-side subscription is toggleable — see "Live subscription is toggleable" below.

## Architecture notes that are not obvious from reading one file

**Sessions can span multiple cwds.** A single session UUID can have events with different `cwd` values (the user `cd`'d mid-session). This is why:
- `Queries.listSessions(cwd)` is derived from the `events` table (`WHERE e.cwd = ?`), not from `sessions.cwd`. Adding cwd-scoped queries that join through `sessions.cwd` will silently miss sessions.
- `upsertSession`'s `ON CONFLICT` does `cwd = COALESCE(excluded.cwd, sessions.cwd)` — last-event cwd wins. Don't flip this back to `COALESCE(sessions.cwd, excluded.cwd)`; that pins a session to its first cwd and breaks `listSessions` for any project the session moved into.

**Tool calls and tool results are separate tables.** `tool_calls` (assistant emits) and `tool_results` (user message replies) join on `tool_use_id`. Latency = `tool_results.ts - tool_calls.ts`. The `events` table also carries `tool_name` / `tool_use_id` / `tool_result_for_id` denormalized for cheap filtering, but the join through the dedicated tables is what powers per-tool latency stats.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omar-toma/ccpulse](https://github.com/omar-toma/ccpulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
