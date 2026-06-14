---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

Clevis is a GitHub analytics dashboard with three independently deployable services and one shared Python library:

- **`apps/api`** — FastAPI REST backend (Python). Handles auth, analytics, RBAC, and job enqueueing. Uses SQLAlchemy 2 + Alembic against PostgreSQL.
- **`apps/worker`** — Standalone Python process. Polls the `jobs` table using `SELECT … FOR UPDATE SKIP LOCKED` and calls the GitHub API to execute background tasks (currently: clearing Actions cache). Uses raw psycopg3, not SQLAlchemy.
- **`apps/ui`** — Next.js 15 / React 19 frontend. Uses TanStack Query for data fetching, Tailwind v4, Base UI primitives, shadcn components.
- **`packages/checks`** — `clevis-checks` Python package. Defines `Check` base class and GitHub security checks (MFA, branch protection, secret scanning). Must be installed editable for `import checks` to work.

### Database models (`apps/api/src/core/db.py`)

Three tables managed by Alembic — no runtime DDL:
- **`github_installations`** — account login, installation ID, auth mode, and `token_ref` (a symbolic reference like `tok_acme`, not the actual token).
- **`audit_logs`** — immutable audit trail; every significant action (cache clear, dry-run, etc.) writes here with actor, action, target, and payload JSON.
- **`jobs`** — job queue; composite index on `(status, job_type)` for efficient worker polling. Status lifecycle: `queued → processing → done/failed`. The `result` column stores JSON on success or a raw exception string on failure.

### Job queue flow

**Enqueueing (API):** `POST /repos/{owner}/{repo}/actions-caches/clear` — if `dry_run=true`, only an audit log is written; if `dry_run=false`, the token is Fernet-encrypted and a `jobs` row with `status='queued'` is inserted.

**Processing (worker):** atomic `SELECT FOR UPDATE SKIP LOCKED` claims one job, updates status to `processing`, decrypts the token, calls the GitHub API, then updates to `done` or `failed`. Multiple worker replicas can poll safely.

### RBAC

Role passed via `X-Role` HTTP header (`viewer` / `analyst` / `admin`). Defaults to `settings.default_rbac_role`. Enforced via `require_role()` FastAPI dependency (`src/services/rbac.py`). Only the cache-clear endpoint requires `admin`; all other routes are unrestricted. There is no session or JWT — the header is trusted to come from an auth proxy.

### Token encryption

Tokens are never stored persistently. When a job is enqueued the API encrypts the token with Fernet (key derived via SHA-256 of `JOB_SECRET_KEY`, base64-encoded). The worker decrypts it at processing time. Both sides duplicate the same `_crypto.py` logic.

### Database URL dialect

The API uses `postgresql+psycopg://` (SQLAlchemy dialect). The worker strips the `+psycopg` prefix to obtain a plain `postgresql://` URL for psycopg3's native `connect()`.

### Analytics & checks integration

`analytics_service.py` calls `checks.runner.run_all_checks(owner, token, base_url)`, which instantiates and runs the three `Check` subclasses in `packages/checks`. Score is computed as `100 - (failed_count / total_count * 100)`. Each check returns `{"status": "pass"|"fail", "value": <object>}`. The checks package handles GitHub API pagination internally via Link header parsing — callers receive complete results.

### Request ID propagation

`RequestIdMiddleware` (`apps/api/src/core/middleware.py`) assigns a UUID to every request (or forwards `X-Request-ID` if present), stores it in a `ContextVar`, and injects it into all log records. The ID is echoed in the `X-Request-ID` response header.

### UI routing

Owner and repo are joined with `~` in URL dynamic segments (e.g., `/repos/octocat~hello-world/cache`). The API client lives in `apps/ui/lib/api/client.ts`; it centralises JSON serialisation, error parsing, and `X-Role` header injection.

## Development setup

```bash
# One-time setup
cp .env.example .env           # fill in ALL variables — none have defaults
pip install -r apps/api/requirements.txt
pip install -r requirements-test.txt
pip install -e packages/checks
cd apps/ui && npm install
```

**Required env vars (6 total):** `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `JOB_SECRET_KEY`, `AUTH_SECRET`, `NEXT_PUBLIC_API_BASE`. Two more deploy-time vars are optional (safe defaults in code): `CORS_ORIGINS`, `GITHUB_API_BASE`. Everything else lives in the `app_config` DB table (configured via Settings page).

Key variables:
- `DB_USER`, `DB_PASSWORD`, `DB_NAME` — Postgres credentials. Docker Compose maps these to `POSTGRES_USER/PASSWORD/DB` for the db container; entrypoints construct `DATABASE_URL` from them (host = `db`).
- `DATABASE_URL` — local dev only (outside Docker); format: `postgresql+psycopg://<user>:<pass>@localhost:5432/<db>`.
- `JOB_SECRET_KEY` — Fernet key for token encryption; generate with `openssl rand -hex 32`
- `AUTH_SECRET` — JWT signing secret; generate with `openssl rand -hex 32`
- `NEXT_PUBLIC_API_BASE` — `http://localhost:8080` for local dev
- `API_PORT` / `UI_PORT` — `8080` / `3000`

**Deploy-time config (env vars, safe defaults in code):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nazarli-shabnam/clevis](https://github.com/nazarli-shabnam/clevis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
