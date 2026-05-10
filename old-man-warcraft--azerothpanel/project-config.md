---
trigger: always_on
description: Trust these instructions fully. Only search the codebase when you need detail not covered here.
---

# AzerothPanel – Copilot Coding Agent Instructions

Trust these instructions fully. Only search the codebase when you need detail not covered here.

---

## What the Repository Does

AzerothPanel is a web-based management panel for [AzerothCore](https://www.azerothcore.org/) WoW private servers. It lets administrators start/stop servers, manage players, tail logs in real time, run GM commands via SOAP, browse and query databases, trigger AzerothCore CMake builds, run the installer, manage modules, edit config files, and extract client data — all from a React UI backed by a FastAPI REST + WebSocket API.

---

## Stack & Runtimes

| Layer | Technology |
|---|---|
| **Frontend** | React 18, TypeScript 5.7, Vite 6, Tailwind CSS 3, Zustand 5, TanStack React Query v5, axios, Monaco Editor |
| **Backend** | Python 3.12, FastAPI 0.115.5, uvicorn, SQLAlchemy 2 (async), aiosqlite (SQLite panel.db), aiomysql (game DBs) |
| **Serving** | nginx (static files + reverse proxy), Docker Compose v2, `network_mode: host` for backend |
| **Host daemon** | `backend/ac_host_daemon.py` — runs **outside Docker** on the host, owns AzerothCore process PIDs |

---

## Project Layout

```
.                          Root: Makefile, docker-compose.yml, nginx.conf, .env.example
backend/                   FastAPI app
  ac_host_daemon.py        Host-side daemon (TCP socket, not inside Docker)
  requirements.txt         All Python dependencies (pinned)
  .env.example             Template — copy to backend/.env before running
  app/
    main.py                App factory: FastAPI instance, CORS, lifespan hooks, logging
    core/
      config.py            Pydantic BaseSettings (reads backend/.env)
      database.py          Async SQLAlchemy engine, SQLite init, run_panel_db_migrations()
      security.py          JWT create/verify, password hashing, get_current_user dep
    models/
      panel_models.py      SQLAlchemy ORM models (PanelSettings, WorldServerInstance, …)
      schemas.py           Pydantic request/response schemas
    api/v1/
      router.py            Aggregates all endpoint routers under /api/v1
      endpoints/           One file per feature: auth, server, instances, logs, players,
                           database, installation, compilation, settings,
                           data_extraction, modules, configs
    api/websockets/
      logs.py              WebSocket handler for live log tailing (/ws/logs)
    services/
      panel_settings.py    CRUD helpers for key/value panel settings
      azerothcore/
        server_manager.py  Daemon-aware process start/stop (falls back to subprocess)
        compiler.py        CMake/make runner, yields output lines
        installer.py       AzerothCore installation steps
        soap_client.py     HTTP SOAP client (httpx)
        module_manager.py  Git-based module install/remove
frontend/
  src/
    App.tsx                Route definitions (React Router v6)
    services/api.ts        axios API client (all HTTP calls go here)
    store/index.ts         Zustand global store (auth token, server status)
    types/index.ts         Shared TypeScript type definitions
    pages/                 One .tsx file per panel page
    components/            layout/ (Header, Sidebar, Layout) and ui/ (Button, Card, …)
  vite.config.ts           Dev proxy: /api → :8000, /ws → ws://localhost:8000
  tsconfig.json            strict mode; `@` alias maps to frontend/src/
docs/
  development.md           Full local dev guide (authoritative)
  configuration.md         All env vars and in-panel settings reference
  api.md                   REST API endpoint reference
```

---

## Environment Setup (Required Before Any Command)

1. **Root `.env`** (Docker vars):
   ```bash
   cp .env.example .env   # adjust AC_PATH, CLIENT_PATH, PANEL_PORT if needed
   ```
2. **Backend `.env`** (app secrets):
   ```bash
   cp backend/.env.example backend/.env
   # At minimum set: SECRET_KEY, PANEL_ADMIN_USER, PANEL_ADMIN_PASSWORD
   ```
   Generate a secret key: `openssl rand -hex 32`

Neither `.env` file is committed. Always copy from `.env.example` first.

---

## Building & Validating Changes

### Backend — install & run

Always use the virtualenv inside `backend/.venv`:

```bash
cd backend
python -m venv .venv                       # create venv (once)
source .venv/bin/activate
pip install -r requirements.txt            # install pinned deps

uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Verify Python syntax without starting the server:
```bash
cd backend && source .venv/bin/activate
python -m py_compile app/main.py app/core/config.py app/models/panel_models.py
```

When adding a Python dependency:
1. Activate the venv first: `source backend/.venv/bin/activate`
2. `pip install <package>`
3. Freeze: `pip freeze > backend/requirements.txt`

### Frontend — install, type-check, build

Always run `npm install` before building or type-checking:

```bash
cd frontend
npm install
npx tsc --noEmit          # type-check — must pass with zero errors (validated ✓)
npm run build             # production build: tsc + vite build (validated ✓)
npm run dev               # dev server on :5173 with HMR
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Old-Man-Warcraft/AzerothPanel](https://github.com/Old-Man-Warcraft/AzerothPanel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
