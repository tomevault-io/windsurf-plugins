---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Agetor is a local desktop app that orchestrates CLI coding agents (Claude Code, OpenAI Codex, others) from a kanban board. Each task is a prompt + working directory + agent choice; running it spawns the agent as a child process, streams its stdout/stderr to the UI, and moves the card through columns based on exit status.

It is **Electrobun** (not Electron) — native webviews driven by a Bun main process. Do not reach for Electron APIs, IPC patterns, or Node-only modules.

## Stack and architecture

Two processes share this repo and a small shared types directory:

- **Bun main process** (`src/bun/`) owns the Electrobun `BrowserWindow`, a SQLite store, and an HTTP API the webview talks to. The webview is loaded either from the Vite dev server (`http://localhost:5173` when present) or from the bundled `views://mainview/index.html`.
- **React webview** (`src/mainview/`) renders the kanban board with dnd-kit, talks to the Bun side over `fetch` + SSE, and uses hand-rolled shadcn/ui primitives styled with Tailwind v3 + CSS variables.
- **Shared** (`src/shared/types.ts`) is the only place both processes import from. Keep it free of runtime imports from either side.

The browser ↔ main connection is intentionally a localhost HTTP API (`Bun.serve` on `AGETOR_API_PORT`, default `4317`), not Electrobun's RPC. **The API binds to `127.0.0.1` only** and gates every route (except `/health`) on a **per-launch random token** generated in `src/bun/server.ts:API_TOKEN`. Both the port and the token are passed to the webview via `#api=…&token=…` on the window URL (hash fragment, **not** query string — the bundled `views://` scheme handler treats anything after the scheme as a literal file path and would otherwise look for a file named `mainview/index.html?api=…`). `src/mainview/lib/api.ts` reads them at load and echoes back as `Authorization: Bearer …` on fetches and as `?token=…` on the SSE URL (EventSource can't set headers). A site the user happens to visit can't read the token, so even with `ACAO` set permissively, drive-by CSRF can't drive an agent run.

### Orchestration flow

1. UI calls `POST /tasks` → `orchestrator.createTask` (async) → row in `tasks` table (column `backlog`). `isolation` defaults to `"worktree"`. When isolation is on and `workdir` is a git repo, `createTask` **resolves the base ref to a sha at create time** and pins it on the task row. Default base is `HEAD`; an explicit `baseRef` ("main", "v1.2.3", a sha…) is honored and validated — bad refs return `{ error }` instead of inserting. This pinning is what makes re-runs reproducible: the worktree is always built off the same starting commit even after the source repo moves.
2. UI calls `POST /tasks/:id/start` → `orchestrator.startTask`:
   - **Pre-flight 1 — agent availability** (`agent-status.ts`): if the agent binary isn't on `PATH`, returns a friendly error with an install hint *before* any state mutation.
   - **Pre-flight 2 — workdir isolation** (`worktree.ts`, `prepareWorkdir`): if `task.isolation === "worktree"` and `workdir` is inside a git repo, creates `~/.agetor/worktrees/<task-id>/` on a fresh branch `agetor/<short-id>-<slug>` off the current HEAD. Idempotent — reused across re-runs. Falls back to running in `workdir` if isolation is off or the dir isn't a git repo.
   - Inserts a `runs` row, flips the task to column `running`, persists `branch` + `worktreePath` on the task row, sets `task.runId`.
   - `agents.spawnAgent` calls `Bun.spawn` with the command from `buildCommand(agent, prompt)` and the prepared cwd (worktree path, or raw workdir on fallback).
   - Every stdout/stderr chunk is appended to `run_events` **and** broadcast to all SSE subscribers.
   - On exit: status row updated, task moves to `review` (exit 0) or back to `ready` (non-zero).
3. UI subscribes via `EventSource` on `/runs/:id/events`. The endpoint replays persisted events first, then streams live ones — this is what lets you close and reopen the run panel without losing scrollback.
4. `DELETE /tasks/:id` → `orchestrator.deleteTask` kills any active run, then `removeWorktree` best-effort tears down `git worktree remove --force` + `git branch -D`. If the worktree path still exists afterwards and lives under `dataDir/worktrees/` (our owned namespace), `removeWorktree` does an `rm -rf` fallback — this catches the case where the user changed `task.workdir` after the worktree was materialized, so git in the new workdir doesn't know about the registration. Never blocks the delete.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alamops/agetor](https://github.com/alamops/agetor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
