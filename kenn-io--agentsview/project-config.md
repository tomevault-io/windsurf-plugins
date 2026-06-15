---
trigger: always_on
description: Instructions for autonomous coding agents working in this repository.
---

# AGENTS.md

Instructions for autonomous coding agents working in this repository.

## Scope

- Applies to all agent-driven work in this repo.
- If multiple instruction files exist, follow the most specific one for the
  files you are editing.

## Required Git Rules

1. Commit every turn.
1. Do not amend commits.
1. Do not change branches without explicit user permission.

## Commit Expectations

- Keep commits focused and related to the requested task.
- Use clear conventional commit messages.
- Do not push, pull, or rebase unless explicitly requested.
- Do not include generated-with lines, attribution blocks, validation footers,
  or command transcripts in commit messages.

## Validation

- Run relevant tests before committing when practical.
- If tests cannot be run, state that clearly in the handoff.
- After Go code changes, run `go fmt ./...` and `go vet ./...` before
  committing.

## Backend Parity

- Preserve behavior and query-shape parity between supported storage backends
  whenever practical. SQLite and PostgreSQL/Cockroach queries, indexes,
  aggregations, filtering, and ordering should match until there is a concrete,
  documented reason for them to differ.
- Do not implement a performance or correctness fix for only one backend and
  call the problem solved unless the user explicitly scopes the work to that
  backend, for example "this is only for PostgreSQL". If one backend needs a
  different implementation, explain why and keep the observable behavior the
  same.

## Test Style

- Go tests use `github.com/stretchr/testify` for assertions. Use `require.X`
  when a failed check should abort the test (setup, nil receivers, length checks
  before indexing) and `assert.X` for independent checks that should keep
  running. Don't write `if got != want { t.Fatalf(...) }` in new tests.
- Domain-specific helpers are fine, but they must use testify internally rather
  than stdlib comparisons.

## Safety

- Do not revert user-authored or unrelated local changes unless explicitly
  requested.
- Avoid destructive git commands unless explicitly requested.
- The SQLite database is a persistent archive. Never delete, drop, truncate, or
  recreate it to handle data version changes. Schema changes use non-destructive
  migrations such as `ALTER TABLE` and `UPDATE`; parser changes trigger a full
  resync that builds a fresh DB, syncs files, copies orphaned sessions from the
  old DB, and swaps atomically. Existing session data must be preserved even
  when source files no longer exist on disk.

## Project Overview

agentsview is a local web viewer for AI agent sessions. It syncs session data
from disk into SQLite with FTS5 full-text search, serves a Svelte 5 SPA via an
embedded Go HTTP server, and provides real-time updates via SSE. See
`internal/parser/types.go` for the full list of supported agents.

## Architecture

```text
CLI (agentsview) -> Config -> DB (SQLite/FTS5)
                  |           |
                  v           v
              File Watcher -> Sync Engine -> Parsers (per agent)
                  |           |
                  v           v
              HTTP Server -> REST API + SSE + Embedded SPA
                              |
                              v
                           PG Push Sync -> PostgreSQL (optional)
                              ^
                              |
              HTTP Server (pg serve) <- PostgreSQL
```

- Server: HTTP server with auto-port discovery, defaulting to 8080.
- Storage: SQLite with WAL mode, FTS5 for full-text search, and optional
  PostgreSQL for multi-machine shared access.
- Sync: file watcher plus periodic sync every 15 minutes for session
  directories.
- PG sync: on-demand push sync from SQLite to PostgreSQL via `pg push`.
- Frontend: Svelte 5 SPA embedded in the Go binary at build time.
- Config: `AGENTSVIEW_DATA_DIR` plus per-agent directory overrides and CLI
  flags. Per-agent env vars are listed on each entry in
  `internal/parser/types.go`.

## Project Structure

- `cmd/agentsview/` - Go server entrypoint.
- `cmd/testfixture/` - Test data generator for E2E tests.
- `internal/config/` - Config loading, JSON migration, and flag registration.
- `internal/db/` - SQLite sessions, messages, search, analytics, and schema.
- `internal/postgres/` - PostgreSQL push sync, read-only store, schema, and
  connection helpers.
- `internal/parser/` - Per-agent session file parsers and content extraction.
- `internal/server/` - HTTP handlers, SSE, middleware, search, and export.
- `internal/sync/` - Sync engine, file watcher, discovery, and hashing.
- `internal/timeutil/` - Time parsing utilities.
- `internal/web/` - Embedded frontend copied from `frontend/dist/` at build
  time.
- `frontend/` - Svelte 5 SPA with Vite and TypeScript.
- `scripts/` - Utility scripts for E2E server setup and changelog work.

## Key Files

| Path                             | Purpose                                       |
| -------------------------------- | --------------------------------------------- |
| `cmd/agentsview/main.go`         | CLI entry point, server startup, file watcher |
| `cmd/agentsview/pg.go`           | `pg` command group: push, status, serve       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenn-io/agentsview](https://github.com/kenn-io/agentsview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
