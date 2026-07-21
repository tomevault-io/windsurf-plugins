---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

ServerKit is a server control panel for managing web applications, databases, Docker containers, and security on VPS/dedicated servers. Flask backend (Python 3.11+), React frontend (Vite + SCSS), SQLite/PostgreSQL database, real-time updates via Socket.IO.

## Development Commands

```bash
# Backend (launcher default port 47927, hot-reload)
cd backend && python -m venv venv && source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python run.py

# Frontend (launcher default port 41921, Vite HMR)
cd frontend && npm install && npm run dev

# Both at once (Linux/WSL)
./dev.sh

# Frontend lint
cd frontend && npm run lint

# Frontend production build
cd frontend && npm run build

# Backend tests
cd backend && pytest
cd backend && pytest --cov=app

# Docker
docker compose -f docker-compose.dev.yml up --build
```

Default dev credentials: `admin` / `admin`

## Architecture

### Backend (`backend/`)

Flask app factory in `app/__init__.py` using `create_app()`. Three-layer architecture:

- **`app/api/`** — Flask Blueprints, one file per feature (90+ files). All routes prefixed `/api/v1/`. JWT-protected via `@jwt_required()`.
- **`app/services/`** — Business logic (165+ files). Services are stateless modules called by API routes. Heavy lifting (shell commands, Docker API, file operations) happens here.
- **`app/models/`** — SQLAlchemy ORM models (80+ files). Schema is managed by Alembic migrations in `backend/migrations/versions/` — add a migration for every schema change; the app does not call `db.create_all()`.

Other backend components:
- `app/sockets.py` — Socket.IO event handlers for real-time metrics, logs, terminal
- `app/agent_gateway.py` — Multi-server agent communication
- `app/middleware/security.py` — Security headers middleware
- `config.py` — Environment-based config (development/production/testing)
- `run.py` — Entry point

### Frontend (`frontend/src/`)

React 18 SPA with client-side routing:

- **`pages/`** — Route-level components (60+ files). Each maps to a route in `App.jsx`.
- **`components/`** — Reusable UI components shared across pages.
- **`contexts/`** — React Context providers: `AuthContext` (JWT auth + token refresh), `ThemeContext`, `ToastContext`, `ResourceTierContext` (feature gating).
- **`services/api.js`** — Centralized `ApiService` class handling all HTTP requests, token management, and auto-refresh.
- **`hooks/`** — Custom React hooks for reusable logic.
- **`styles/`** — SCSS stylesheets with design system variables. Main entry is `main.scss`. Page-specific styles in `styles/pages/`.
- **`layouts/`** — `DashboardLayout` wraps authenticated pages (sidebar + header).

Route guards: `PrivateRoute` (auth check), `PublicRoute` (redirect if logged in), `SetupRoute` (redirect to `/setup` if not configured).

### Request Flow

Browser → Nginx (`:80`/`:443`) → proxy_pass to Docker containers (`:8001-8999`) for managed apps, or to Flask (`:5000`) for the panel API. The 404 handler in Flask serves `index.html` for SPA client-side routing; API routes return JSON errors.

### Production Build

The Dockerfile is multi-stage: Node 20 builds frontend, Python 3.11 serves everything via Gunicorn with a single plain threaded worker (`--workers 1 --threads 100`, no `--worker-class` — see the single-worker note above). Built frontend is served from Flask's static folder.

## Platform & Distro Awareness

ServerKit deploys on Linux (bare metal, VPS, or Docker). Development may happen on Windows/macOS.

- **Service layer is Linux-only** — nginx, systemctl, apt/dnf, PHP-FPM, etc. are inherently Linux. No need to abstract these for Windows.
- **Platform-agnostic code** (config management, storage, API layer) should guard Unix-only calls like `os.chmod` with `if os.name != 'nt'` so the dev server can run locally on any OS.
- **Distro differences matter** — use `backend/app/utils/system.py` helpers (`get_package_manager`, `is_package_installed`, `install_package`) instead of calling `apt`/`dpkg`/`dnf` directly. Not all targets are Debian-based.

## Code Style

### Python
- PEP 8, type hints where helpful
- Service functions are standalone (no classes unless stateful)
- Consistent JSON error responses: `{'error': 'message'}, status_code`

### React/JavaScript
- Functional components with hooks only
- PascalCase for components (`Sidebar.jsx`), camelCase for everything else
- SCSS for styling — use existing design system variables (`$bg-card`, `$primary-color`, `$spacing-md`, etc.) and BEM-like naming (`.block__element--modifier`)
- Context API for global state; props drilling is fine for 2-3 levels
- No inline styles; no Tailwind/CSS-in-JS

### Diffs & Commits
- One logical change per commit
- Minimal, focused diffs — don't silently refactor surrounding code
- Branch naming: `feature/`, `fix/`, `docs/`, `refactor/` prefixes
- **Pre-commit hook**: a shared git hook (`.githooks/pre-commit`) lints staged
  frontend JS/JSX and blocks the commit on ESLint **errors** (warnings allowed;
  backend/SCSS-only commits are skipped). Enable once per clone:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhd3197/ServerKit](https://github.com/jhd3197/ServerKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
