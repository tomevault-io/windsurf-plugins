---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Codex Deck is a full-stack TypeScript application providing a web UI for browsing and interacting with Codex CLI conversation history stored in `~/.codex/`. It supports both read-only history browsing and interactive Codex workflows (create threads, send messages, plan mode).

Remote-server adoption is staged. The `server/` and `wire/` workspace packages are the foundation for the future remote codex-deck `server`; use `docs/SERVER_IMPLEMENTATION.md` before making architectural changes there.

## Commands

Package manager is **pnpm**. Node.js 20+ required. This is a pnpm workspace: root `api/`+`web/` is the local app; `server/` and `wire/` are workspace packages.

### Root (local app)

- `pnpm dev` — run frontend (port 12000) + backend (port 12001) concurrently with watch mode
- `pnpm dev:web` — Vite dev server only (proxies `/api/*` to :12001)
- `pnpm dev:server` — backend only via `tsx watch api/index.ts --dev`
- `pnpm build` — production build (builds `wire` first, then server via tsup + web via Vite)
- `pnpm test` / `pnpm test:unit` — run the root unit test suite under `tests/unit/`
- `pnpm start` — run built CLI from `dist/index.js`
- `pnpm exec prettier --write .` — format before PRs

Run a single test file: `pnpm exec tsx --tsconfig tsconfig.test.json --test tests/unit/<file>.test.ts`

### Remote packages

- `pnpm --dir wire build` / `pnpm --dir wire test` — build/test the shared `@zuoyehaoduoa/wire` package
- `pnpm --dir server dev` — start the remote server with `.env` + `.env.dev`
- `pnpm --dir server build` — typecheck the remote server package
- `pnpm --dir server test` — run remote server tests
- `docker build -f Dockerfile.server .` — build the remote-server container image

### Per-surface test guidance

Run the tests that match the surface you changed:

- `api/` or `web/` changes → `pnpm test` and `pnpm build`
- `wire/` changes → `pnpm --dir wire test`
- `server/` changes → `pnpm --dir server test` and `pnpm --dir server build`
- Cross-cutting remote/auth changes → run all of the above

## Architecture

### Backend (`api/`)

Hono framework on Node.js HTTP server. Key modules:

- **`index.ts`** — CLI entry point (Commander.js). Parses `--port`, `--dir`, `--dev`, `--no-open`.
- **`server.ts`** — All Hono routes: session listing, conversation fetching, file diffs/tree/content, terminal runs, SSE streaming, and interactive Codex thread management. Also serves built frontend assets from `dist/web/`.
- **`storage.ts`** — Reads/parses Codex session data from `~/.codex/` (history.jsonl + per-session .jsonl files). Exports all shared TypeScript interfaces used by both backend and frontend.
- **`watcher.ts`** — Chokidar-based filesystem watcher with 20ms debounce. Monitors history.jsonl and sessions/ directory, fans out change events for SSE.
- **`codex-app-server.ts`** — Bridges web UI to `codex app-server` subprocess via RPC for interactive mode (create threads, send messages, interrupt, handle user-input prompts).
- **`path-utils.ts`** — Backend path normalization.

### Remote Server Adoption (`server/`, `wire/`)

- **`server/`** — Staged codex-deck remote server package and browser entrypoint for remote mode.
- **`wire/`** — Canonical shared protocol package for encrypted remote/server payloads, OPAQUE auth helpers, and session/update schemas.

Important: `server/` is not the current production backend for local codex-deck. It is a migration/adoption target. Prefer documenting and making deliberate architectural changes there rather than assuming it already matches codex-deck behavior.

When working inside `server/`, also read `server/CLAUDE.md` for package-specific guidance that applies to that subtree.

### Frontend (`web/`)

React 19 SPA built with Vite 6 + Tailwind CSS 4.

- **`app.tsx`** — Main orchestrator: three-pane layout (session list | conversation | diff/files), local/remote connection mode, message composer, model/plan mode selection, and live-update integration.
- **`components/`** — Session list, session view, diff pane, message block, markdown renderer, and `tool-renderers/` for specific tool output (bash, edit, read, search, tasks, etc.).
- **`api.ts`** — Browser API wrapper. Chooses local vs remote transport, exposes remote login/admin helpers, and notifies conversation subscribers after mutating calls.
- **`transport/`** — Unified browser transport abstraction. `local.ts` uses EventSource against `/api/*`; `remote.ts` adapts the same UI to a remote CLI via encrypted polling.
- **`remote-client.ts`** — Browser-side remote auth/session client. Handles OPAQUE-based login, encrypted relay RPCs, saved remote login restore, and remote machine selection.
- **`hooks/`** — `use-event-source.ts` (SSE client), `use-page-visibility.ts` (tab visibility).

### Shared Types

Both backend and frontend import local app types from `api/storage.ts` via the path alias `@codex-deck/api` (configured in Vite and tsconfig). This is the single source of truth for interfaces like `Session`, `ConversationMessage`, `CodexThreadStateResponse`, etc.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asfsdsf/codex-deck](https://github.com/asfsdsf/codex-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
