---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

MultiTable is a local, browser-based dashboard for managing AI coding agents and dev processes. A Node.js daemon spawns PTYs via `node-pty`, persists state in SQLite, and serves a React UI over REST + WebSocket on `localhost:3000`. See `docs/OVERVIEW.md` and `docs/SPEC.md` for the product concept; this file is about working in the code.

## Monorepo layout

npm workspaces under `packages/*`:

- `packages/daemon` — Node.js + Express + `ws` + `node-pty` + `better-sqlite3`. The entire backend.
- `packages/web` — React + Vite + xterm.js + Zustand + TailwindCSS. Builds into `packages/daemon/dist/public` so the daemon serves the SPA.
- `packages/cli` — the `mt` CLI entrypoint (commander).

## Commands

Run from the repo root unless noted.

```bash
npm install                      # installs all workspaces
npm run dev                      # concurrently runs daemon (tsx watch) + web (vite dev)
npm run build                    # builds all workspaces
npm run lint                     # eslint packages/*/src --ext .ts,.tsx
npm run format                   # prettier --write packages/*/src
```

Per-workspace:

```bash
# daemon
npm run dev   -w @multitable/daemon    # tsx watch src/index.ts
npm run build -w @multitable/daemon    # tsc + copies src/db/schema.sql → dist/db/schema.sql
npm run start -w @multitable/daemon    # node dist/index.js

# web
npm run dev   -w @multitable/web       # Vite dev server, proxies /api and /ws to :3000
npm run build -w @multitable/web       # tsc + vite build → packages/daemon/dist/public

# cli
npm run build -w @multitable/cli
```

The daemon listens on `http://127.0.0.1:3000` and exposes `ws://127.0.0.1:3000/ws`. In dev, Vite proxies both to the daemon — start the daemon before (or alongside) the web dev server.

No test framework is configured yet — do not invent `npm test` incantations.

## Build gotcha: schema.sql

`tsc` does not copy non-TS assets. The daemon's `build` script already runs `cp src/db/schema.sql dist/db/schema.sql`, but if you ever invoke `tsc` directly (e.g. `npx tsc` inside `packages/daemon`), you must also copy the schema or the daemon will crash on startup when it tries to init the DB.

## Architecture

### Daemon (`packages/daemon/src`)

Startup sequence lives in `index.ts` and is load-bearing — read it before changing boot order:

1. Load global config (`config/loader.ts`, reads `~/.config/multitable/config.yml` via `env-paths`).
2. Check `pids.json` for orphaned processes from prior runs (`pids.ts`).
3. Init SQLite (`db/store.ts` — schema from `db/schema.sql`).
4. Create `PtyManager` and `PermissionManager`.
5. Install Claude Code hooks into each project's `.claude/settings.json` (`hooks/installer.ts`).
6. Build Express + WS server (`server.ts`).
7. Autostart sessions/commands from the DB; attach file watchers.
8. Listen on `host:port`.

Key modules:

- `pty/manager.ts` — the single source of truth for live processes. Holds an in-memory `ManagedProcess` map keyed by process id, handles spawn / restart / exit / auto-restart backoff, emits events (`state-changed`, `metrics`, `exit`, `resume-failed`) consumed by `server.ts`.
- `pty/ringBuffer.ts` — per-process scrollback buffer replayed to new WS subscribers.
- `pty/stream.ts` — the WS message router (`handleWsMessage`), translates `pty-input`, `subscribe`, `unsubscribe`, resize, etc. into manager calls.
- `db/store.ts` — better-sqlite3, synchronous. Exported functions are the DB API; routers call them directly rather than going through a service layer.
- `api/*.ts` — one router per resource (`projects`, `sessions`, `commands`, `terminals`, `processes`, `config`, `search`, `transcripts`, `notes`). Each is a factory that receives the `PtyManager` and returns an `express.Router`.
- `hooks/` — Claude Code integration. `installer.ts` writes hook entries into project `.claude/settings.json` so Claude Code calls back into `/api/hooks/*`. `receiver.ts` is the HTTP receiver. `permissionManager.ts` holds pending permission prompts until the UI resolves them; `costParser.ts`, `labeler.ts`, `optionDetector.ts`, `promptsParser.ts` parse hook payloads.
- `watcher/index.ts` — chokidar-based file watcher for `mt.yml` changes and per-process `fileWatchPatterns` restart triggers.
- `tracker/`, `git/`, `conflict/` — cost tracking, git diff via `simple-git`, and process-conflict detection.
- `types.ts` — shared types (`ManagedProcess`, `ProcessState`, `WsMessage`, `PermissionPrompt`, `Project`, `GlobalConfig`, `ProjectConfig`, `SpawnConfig`). Import types from here rather than redefining.

### API routing quirk

Creation endpoints `POST /api/projects/:id/{sessions,commands,terminals}` live on the **projects router**, not the resource routers. The projects router calls DB store functions directly. The per-resource routers (`/api/sessions`, `/api/commands`, `/api/terminals`) handle mutations on an existing id (`PUT`, `DELETE`, lifecycle actions). If you're adding a creation route, put it on the projects router to stay consistent.

### WebSocket


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erickalfaro/multitable](https://github.com/erickalfaro/multitable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
