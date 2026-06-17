---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Engram is a disc ripping and media organization tool with a reactive web dashboard. It automates the workflow from optical disc insertion to organized media library, with Human-in-the-Loop intervention for ambiguous content. Cross-platform backend (Python/FastAPI), with automatic drive detection on Windows and Linux. macOS can run the backend and dashboard but requires manual job submission (no automatic drive detection on macOS). Requires MakeMKV with a valid license.

## Important Rules

- **NEVER delete `backend/engram.db`** unless the user explicitly asks. It contains API keys and credentials that must be re-entered manually.
- **Always terminate this session's servers when work is done — and before opening a PR.** Kill the `uvicorn`/`python` (uvicorn workers) and `makemkvcon` processes you started; orphans cause duplicate jobs and MakeMKV drive conflicts. If you're the only running session, killing them all is fine; if other sessions are live, scope the kill to **your own ports** so you don't take down a sibling (see "Parallel sessions / worktree isolation"). Never use `--reload` — it spawns a child process with its own drive sentinel, creating duplicate disc events.

## Repository Organization

Keep the repo root clean. Where things belong:

- **Dated working docs** (plans, specs, reviews) → `docs/superpowers/{plans,specs,reviews}/` named `YYYY-MM-DD-kebab-title.md`. Review screenshots go in `docs/superpowers/reviews/assets/`.
- **Committed UI screenshots** → `docs/screenshots/` (user-facing, numbered) and `docs/design_handoff_synapse/screenshots/` (design handoff). The brand-system handoff lives at `docs/design_handoff_brand/`; screen-level UI direction at `docs/design_handoff_synapse/`.
- **Design explorations / brainstorm HTML** kept for reference → `docs/design_handoff_synapse/explorations/`.
- **Local-only debug screenshots & one-off build artifacts** → gitignored `artifacts/` (never the repo root). The root `/*.png` glob and `artifacts/` are gitignored to keep stray captures out of git.
- **Brand raster assets** are generated, not hand-placed → `frontend/public/brand/` via `npm run brand:export` from SVG sources in `frontend/public/brand/sources/`.

## Commands

### Backend (from `backend/`)

```bash
uv sync                        # Install/sync dependencies
uv run uvicorn app.main:app    # Start dev server (port 8000)
uv run pytest                         # Run all tests
uv run pytest test_file.py::test_name # Run a single test
uv run ruff check .                   # Lint
uv run ruff format .                  # Format
```

### Frontend (from `frontend/`)

```bash
npm install          # Install dependencies
npm run dev          # Start Vite dev server (port 5173)
npm run build        # TypeScript check + production build
npm run lint         # ESLint
npm run test:e2e     # Run Playwright E2E tests
npm run test:e2e:ui  # Run E2E tests with interactive UI
npm run brand:export # Regenerate favicons + .ico/.icns from SVG sources
```

### Parallel sessions / worktree isolation

Running two Claude/dev sessions at once (e.g. two `.claude/worktrees/*`) collides on the
default **ports** (backend `8000`, Vite `5173`) and on any **shared database**. Give each
session its own ports + DB:

| Knob | Env var | Default | Notes |
|------|---------|---------|-------|
| Backend DB | `DATABASE_URL` | `sqlite+aiosqlite:///./engram.db` | Read at import (`database.py`). Each worktree's relative `./engram.db` is already distinct — only override if you pointed it at a **shared/real** DB. Never let two live sessions share one DB file. |
| Backend port | uvicorn `--port` flag | `8000` | **`PORT`/`HOST` env vars are ignored** by `uvicorn app.main:app` (only honored by `python -m app.main`). Use the CLI flag. |
| Frontend port | `VITE_PORT` | `5173` | Vite dev server. |
| Proxy target | `VITE_BACKEND_PORT` | `8000` | **Must equal the backend `--port`** or `/api` + `/ws` 502. |

Second stack (PowerShell), backend from `backend/`, frontend from `frontend/`:

```powershell
# backend  (distinct DB + port)
$env:DATABASE_URL = "sqlite+aiosqlite:///./engram-b.db"
uv run uvicorn app.main:app --port 8100

# frontend (distinct port; proxy points at the backend above)
$env:VITE_PORT = "5273"; $env:VITE_BACKEND_PORT = "8100"; npm run dev
```

bash equivalent: `DATABASE_URL=sqlite+aiosqlite:///./engram-b.db uv run uvicorn app.main:app --port 8100`
and `VITE_PORT=5273 VITE_BACKEND_PORT=8100 npm run dev`.

**Drive sentinel is per-backend and unconditional** (`job_manager.start()` → `_drive_monitor.start()`):
every backend polls the physical optical drive. Two live backends → duplicate disc events +
`makemkvcon` conflicts. For parallel work use **simulation** (`DEBUG=true` + `/api/simulate/*`),
never two backends against a real inserted disc. Real-disc testing = exactly one backend.

**Clean up before the PR.** When the work is done, stop the servers this session started —
scoped by port so a sibling session keeps running:

```powershell
# Stop THIS session's backend + frontend by port (use the ports you launched on)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jsakkos/engram](https://github.com/Jsakkos/engram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
