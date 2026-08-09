---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Flask application, split across `healthcheck.py` plus a handful of focused modules
(`dbstore.py`, `auth.py`, `poller.py`, `admin.py`), that exposes health-check endpoints (JSON) plus a React
dashboard for monitoring device online status, key expiry, and update status across a tailnet. A background
poller refreshes devices/tailnet keys from the Tailscale API into a SQLite database (`DATABASE_PATH`); the
`/health*` JSON API and dashboard read from that snapshot rather than calling Tailscale per request. A
web-based admin UI (`/admin`) provides a first-run setup wizard, login, settings, user management, and an
audit log, all backed by the same database. Designed to run under Gunicorn in Docker, and to be scraped by
monitoring tools like Gatus.

## Commands

```bash
# Setup
python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt

# Run for local dev (auto-reload)
FLASK_APP=healthcheck.py flask run --port 5000

# Run like production
gunicorn -w 4 -b 0.0.0.0:5000 -c gunicorn_config.py healthcheck:app

# Lint (see .flake8 for max-line-length=140 and ignored rules)
pip install flake8 && flake8 healthcheck.py dbstore.py auth.py poller.py admin.py notifier.py gunicorn_config.py

# Tests
pip install pytest && pytest -q
pytest tests/test_dbstore.py -q          # single file
pytest tests/test_dbstore.py::test_name -q  # single test

# Docker
docker build -t tailscale-healthcheck .
docker run -p 5000:5000 -v tailscale-healthcheck-data:/data --env-file .env tailscale-healthcheck
```

The Python backend has no separate build step. The dashboard/admin UI is a Vite/React SPA under `frontend/`,
built into `static/app/` (done automatically in the multi-stage Dockerfile; for local dev run `pnpm build`
inside `frontend/` after editing it, or `pnpm dev` for hot reload against a locally running Flask backend).

## Architecture

The backend is split across a few modules:

- `healthcheck.py` — Flask app setup, `/health*` JSON API, dashboard shell routes, rate limiting, and
  OAuth token machinery.
- `dbstore.py` — SQLite connection/schema management (WAL mode), settings get/set with env-override
  semantics, device/key snapshot read/write with change diffing, audit log read/purge, user CRUD.
- `auth.py` — Flask-Login wiring (`LoginManager`, `User`), backed by `dbstore`'s users table.
- `poller.py` — background thread that refreshes devices/tailnet keys from the Tailscale API into SQLite
  every `POLL_INTERVAL_SECONDS`; elects a single runner across Gunicorn workers via an `fcntl` lock file.
- `admin.py` — the `/admin` Blueprint: setup wizard, login, settings, user management, audit log (both
  HTML shell routes and a `/admin/api/*` JSON API the React admin pages call).

Key pieces, in the order a change usually touches them:

1. **Config via the settings registry** — Every runtime-configurable setting (connection info, health
   thresholds, device/key filters, timezone, HTTP timeout, log level, rate limiting, retry/backoff, poll
   interval, audit retention, debug log capture) lives in `dbstore.SETTINGS_REGISTRY`, keyed by setting name
   to a 5-tuple `(env_var, type, default, sentinel, group)`. Resolution is env-first, DB-fallback:
   `dbstore.get_setting(name)`/`get_setting_typed(name)` return the env var's value if set (and not equal to
   the sentinel placeholder, e.g. `TAILNET_DOMAIN=example.com`), else the last value saved to the database
   (by `sync_env_settings()` at boot, the setup wizard, or `/admin/settings`), else `default`. This is what
   lets the admin UI change settings without a restart (for most of them — see below) and lets removing an
   env var later fall back to the last-known-good DB value instead of reverting to "unconfigured".
   **When adding a new setting**: add one entry to `SETTINGS_REGISTRY` (that's what makes it show up in
   `/admin/settings`, auto-grouped, and auto-synced from its env var) rather than a standalone
   `os.getenv(...)` call, and document it in `.env.example` + README's Configuration table. `PORT` and the
   Gunicorn bind/worker-count flags are the deliberate exception — pure process-bootstrap concerns, not in
   the registry, env-only.
   **Avoid N+1 settings reads in per-device loops**: `dbstore.get_settings_typed(names)` resolves several
   settings in a single DB round trip — call it once per request/poll-cycle (see `HEALTH_SUMMARY_SETTINGS` /
   `DEVICE_FILTER_SETTINGS` / `UPDATE_HEALTHY_FILTER_SETTINGS` / `KEY_FILTER_SETTINGS` in `healthcheck.py`)
   and pass the resulting dict into per-device helpers like `should_include_device(device, filters)` —
   never call `get_setting_typed()` per device.
   **Restart-required settings**: `RATE_LIMIT_*` and `LOG_LEVEL` are wired up once at process/Flask-app
   startup (Flask-Limiter construction, `logging.basicConfig()`), so saving a new value persists it
   immediately but it only takes effect after a restart — see `admin.RESTART_REQUIRED_SETTINGS`, surfaced to
   the UI via each field's `restart_required` flag in the `/admin/api/settings` response.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laitco/tailscale-healthcheck](https://github.com/laitco/tailscale-healthcheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
