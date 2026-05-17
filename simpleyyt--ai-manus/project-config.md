---
trigger: always_on
description: > Canonical guide for AI coding agents working on the **AI Manus × Claw** codebase.
---

# AGENTS.md

> Canonical guide for AI coding agents working on the **AI Manus × Claw** codebase.

---

## Project Overview

AI Manus × Claw is a general-purpose AI Agent system with an integrated [OpenClaw](https://github.com/anthropics/openclaw) AI assistant, comprising five services:

| Service | Stack | Port (dev) | Entry Point |
|---|---|---|---|
| **Frontend** | Vue 3 + TypeScript, Vite 4, Tailwind CSS | 5173 | `frontend/src/main.ts` |
| **Backend** | Python 3.12, FastAPI, LangChain, Beanie/Motor | 8000 | `backend/app/main.py` |
| **Sandbox** | Python 3.10, FastAPI, Xvfb/Chrome/VNC | 8080 (API), 5900 (VNC) | `sandbox/app/main.py` |
| **Claw** | Node.js, OpenClaw Gateway, manus-claw plugin | 18788 | `claw/entrypoint.sh` |
| **Mockserver** | Python, FastAPI | 8090 | `mockserver/main.py` |

Infrastructure: **MongoDB 7.0**, **Redis 7.0**, **Docker** (sandbox & Claw orchestration).

---

## Directory Structure

```
ai-manus/
├── frontend/          # Vue 3 SPA (Vite, TypeScript, Tailwind)
├── backend/           # FastAPI backend (DDD layout)
│   └── app/
│       ├── domain/           # Models, services, tools, agents, repositories
│       ├── application/      # Application services (auth, agent, file, token, email, claw)
│       ├── infrastructure/   # External integrations (search, browser, sandbox, claw, DB, cache)
│       ├── interfaces/       # API routes, schemas, error handlers, dependencies
│       ├── core/             # Config (config.py)
│       └── main.py
├── sandbox/           # Sandbox service (shell, file, supervisor APIs)
├── claw/              # Claw service (OpenClaw Gateway + manus-claw plugin)
│   └── manus-claw/   # Node.js plugin bridging OpenClaw with Manus backend
├── mockserver/        # Mock LLM server for dev/testing
├── docs/              # Docsify documentation site
├── .cursor/skills/    # Cursor agent skills
├── dev.sh             # Shortcut: docker compose -f docker-compose-development.yml ...
├── run.sh             # Shortcut: docker compose -f docker-compose.yml ...
├── build.sh           # docker buildx bake
├── .env.example       # Environment variable template
├── docker-compose.yml                # Production compose
└── docker-compose-development.yml    # Development compose (hot-reload)
```

---

## Development Environment Setup

### Prerequisites

- **Docker 20.10+** and **Docker Compose**
- **uv** (Python package manager) — for running backend/sandbox outside Docker
- **Node.js / npm** — for running frontend outside Docker
- **Python 3.12+** (backend), **Python 3.10+** (sandbox)

### Quick Start (Docker Compose — Recommended)

```bash
cp .env.example .env
# Edit .env — at minimum set API_KEY to any non-empty string
./dev.sh up -d
```

This starts: frontend (5173), backend (8000), sandbox (8080), mockserver (8090), MongoDB (27017), Redis.

### Key `.env` Values for Development

| Variable | Recommended Value | Purpose |
|---|---|---|
| `AUTH_PROVIDER` | `none` | Skip authentication entirely |
| `API_BASE` | `http://mockserver:8090/v1` | Use mock LLM server |
| `API_KEY` | any non-empty string | Required — set to anything with mockserver |
| `SEARCH_PROVIDER` | `bing_web` | No API key needed |
| `SANDBOX_ADDRESS` | `sandbox` | Use single dev sandbox container |
| `LOG_LEVEL` | `DEBUG` | Verbose logging |

### Running Services Individually (Without Docker)

**Backend:**
```bash
cd backend
uv sync
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```
Requires running MongoDB and Redis. Requires `API_KEY` env var (or `.env` in `backend/`).

**Frontend:**
```bash
cd frontend
npm install
BACKEND_URL=http://localhost:8000 npm run dev
```
The Vite config creates a proxy for `/api` when `BACKEND_URL` is set.

**Sandbox:** Typically Docker-only (requires Xvfb, Chrome, VNC, supervisord).

**Mockserver:**
```bash
cd mockserver
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8090 --reload
```

---

## Testing

### Backend Tests (pytest — integration-style)

Tests live in `backend/tests/` and hit a **running** backend at `http://localhost:8000`.

```bash
# Ensure backend + MongoDB + Redis are running
./dev.sh up -d mongodb redis backend

cd backend
uv run pytest                               # all tests
uv run pytest tests/test_auth_routes.py     # specific file
uv run pytest -m file_api                   # by marker
```

Key test files:
- `tests/test_auth_routes.py` — auth endpoints
- `tests/test_api_file.py` — file upload/download
- `tests/test_sandbox_file.py` — sandbox file operations

Config: `backend/pytest.ini` (`asyncio_mode = auto`, markers: `file_api`).

### Sandbox Tests (pytest)

```bash
./dev.sh up -d sandbox
cd sandbox
uv run pytest
```

### Frontend (No Automated Test Runner)

```bash
cd frontend
npm run type-check    # vue-tsc type checking
npm run build         # production build (catches TS + template errors)
```

For manual UI testing: start full dev stack (`./dev.sh up -d`), open `http://localhost:5173`.

### Mockserver

No tests. Verify with:
```bash
curl -X POST http://localhost:8090/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"mock","messages":[{"role":"user","content":"hi"}]}'
```

### Full-Stack Integration Test

1. `./dev.sh up -d` — start all services

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Simpleyyt/ai-manus](https://github.com/Simpleyyt/ai-manus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
