---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## What This Is

CapForge is a desktop subtitle editor: Electron 33 shell → React 19 renderer → Python FastAPI backend. The backend runs WhisperX for transcription and Pillow/FFmpeg for video rendering. Electron spawns the Python process on startup; the renderer talks to it over REST + WebSocket on `127.0.0.1` (preferred port 53421, falls back to an ephemeral port).

## Git Operations

Delegate all git **write** work — `commit`, `merge`, `push`, `branch`, `rebase`, conflict resolution — to the `git-ops` subagent (`Agent(subagent_type: "git-ops", …)`). It follows conventional-commit format, never commits/pushes directly to the default branch (branches first), and confirms before the first push. The read-only `scout` agent must never be used for git writes. Read-only git inspection (`status`, `diff`, `log`) can be run inline or by `scout`.

## Build & Dev Commands

```bash
npm run dev:react          # Vite dev server with HMR — THE live-reload path
npm run dev                # electron . --dev — does NOT start Vite (see trap below)
npm start                  # Production mode (needs build:react first)
npm run build:react        # Production build → out/renderer/

npm run typecheck          # tsc --noEmit -p tsconfig.web.json
npm test                   # vitest run
npm run lint               # eslint src

npm run backend            # uvicorn backend.main:app --host 127.0.0.1 --port 53421
npm run dist:mac           # DMG   (dist:* run build:react first)
npm run dist:win           # NSIS installer
```

**`npm run dev` is a trap**: it launches Electron with `--dev` but does not start Vite, so renderer edits won't appear. Use `npm run dev:react`.

### Testing

- **Frontend**: vitest, ~465 tests. Runs in the **`node` environment, not jsdom** — component tests render via `react-dom/server` to static HTML and assert on markup, so there are no DOM events and no testing of hooks-with-effects. Write new tests to that constraint.
- **Backend**: pytest. `pyproject.toml` sets `testpaths = ["backend/tests"]`, so a bare `pytest` **silently skips `mcp_server/tests`** — pass that path explicitly.
- **CI** (`.github/workflows/ci.yml`): three jobs — frontend (typecheck + vitest + lint), backend (pytest), caption-parity.
- eslint (flat config) + prettier are configured, but **`eslint.config.js` deliberately ignores `electron/**`** — `npm run lint` passing says nothing about the Electron layer.

## Architecture

### Three-Layer Stack

1. **Electron main process** (`electron/`) — vanilla JS. `main.js` creates the window, `python-manager.js` spawns/manages the backend, `runtime-setup.js` handles first-launch Python/model downloads, `preload.js` bridges IPC.
2. **React renderer** (`src/renderer/src/`) — TypeScript + React 19 + Tailwind v4, built by electron-vite.
3. **Python backend** (`backend/`) — FastAPI on uvicorn. `engine/transcriber.py` runs the WhisperX pipeline (transcribe → align → diarize). `exporters/video_render.py` renders caption frames with Pillow and muxes with FFmpeg. `models/schemas.py` defines all Pydantic models.

### Dual preload gotcha (read before adding any `window.subforge.*` API)

`package.json` `main` is **`electron/main.js`**, and it loads **`electron/preload.js`** (vanilla CJS) as the preload. `src/preload/index.ts` is a **types mirror**: electron-vite compiles it to `out/preload/`, which nothing loads. Likewise `src/main/index.ts` is a stub that `require`s `electron/main.js` — the TS migration is half-finished, and only `out/renderer/` is consumed from the electron-vite build.

So adding an API means **three** edits: `ipcMain.handle` in `electron/main.js`, the bridge in `electron/preload.js` (runtime), and the typed signature in `src/preload/index.ts` (renderer types). Editing only the TypeScript file type-checks and silently does nothing at runtime.

### Communication

- **REST** — `backend/main.py` registers ~45 routes; the ones with non-obvious rules:
  - **Local-token gated** (`require_local_token`, 7 routes): `GET /api/fonts/system`, `GET /api/serve-audio`, `GET /api/video-info`, `PUT /api/result`, `POST /api/export`, `POST /api/render-video`, `POST /api/export-hyperframes`. `GET /api/result` is **ungated** — only the PUT is.
  - **Agent-token gated**: all `/api/agent/*` (~19 routes) plus `POST /api/render-frame`.
  - **Deliberately ungated UI mirrors**: `GET|POST /api/coauthor`, `POST /api/coauthor/sync-captions`, and `PUT /api/ui-state` (the write side of the agent-gated `GET /api/agent/ui-state`). The local renderer has no agent token but drives co-author mode from the HyperFrames panel; same loopback trust level as `/api/export-hyperframes`, and the source comments say so. Don't "fix" these by gating them.
- **WebSocket**: `/ws/progress` pushes `ProgressUpdate` events (status + percentage + message).
- **IPC**: renderer ↔ main via `ipcRenderer.invoke()` / `ipcMain.handle()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FRSname/CapForge](https://github.com/FRSname/CapForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
