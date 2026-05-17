---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

- Django 5.2, Python ≥3.10, DRF (`rest_framework` + `rest_framework.authtoken`). DRF defaults: `TokenAuthentication` + `IsAuthenticated` (`core/settings.py`).
- **Database**: Postgres in Docker / prod (`psycopg`); SQLite fallback for host dev. Selection is implicit in `core/settings.py` — if `POSTGRES_HOST` is set in env it uses Postgres, otherwise it falls back to `db.sqlite3` tuned with `journal_mode=WAL`, `synchronous=NORMAL`, `transaction_mode=IMMEDIATE`, `timeout=30`. Don't assume the dev DB is the prod DB.
- Dep mgmt via `uv` (`pyproject.toml`, `uv.lock`); lint/format via `ruff`. Frontend uses `pnpm`.
- Static: Whitenoise with `CompressedManifestStaticFilesStorage`; Tailwind v4 (`static/input.css` → `static/output.css`).
- Frontend SPA under `frontend/`: Vite + React 19 + TS, **TanStack Router** (file-based, generated `routeTree.gen.ts`) + **TanStack Query**, **Jotai** for state, **shadcn / Radix** + Tailwind v4 for UI. Talks to Django via `/api/v1/`.
- Celery + Redis for async work; `django_celery_beat` (`DatabaseScheduler`) for periodic tasks — schedules edited via Django admin, not code. `CELERY_TASK_ACKS_LATE=True`, `CELERY_TASK_REJECT_ON_WORKER_LOST=True`.
- LLM: `openai` SDK, structured output via `pydantic` (`SkillAssessment`, `RelevanceCheck`, `LinkedInIngest`). Model from `OPENAI_MODEL` (default `gpt-4o-2024-08-06`); cheap relevance gate uses `OPENAI_RELEVANCE_MODEL` (default `gpt-4o-mini`). All OpenAI calls go through `core.openai.get_prompt_manager()`.
- LinkedIn ingestion uses **Apify** (`apify-client`, actor `harvestapi/linkedin-profile-scraper`) — not a custom scraper.
- Auth: `core.auth.EmailBackend` (login by email) **plus** Google OAuth via `/api/v1/auth/google/` (verifies `GOOGLE_OAUTH_CLIENT_ID`).
- Payments: Mayar (`core/payments/mayar.py`) — payment-link create + webhook verified by `X-Callback-Token`.
- Realtime: Redis pub/sub (`core/realtime.py`) → SSE endpoint `/api/v1/subscriptions/stream/` (EventSource auth via `?token=` query param, since EventSource cannot set headers).
- Notifications:
  - Discord webhook (`core/notifications/discord.py`), called from Celery tasks; no-ops when `DISCORD_WEBHOOK_URL` is empty.
  - Transactional email (`core/notifications/email.py:send_email`) using Django's SMTP backend; no-ops with a warning when `EMAIL_HOST` / `EMAIL_HOST_USER` are unset. Daily summary task is `assessment.tasks.email_morning_high_score_summary` (registered via beat); links into the SPA use `FRONTEND_URL`. There's a superuser admin page at `/settings/smtp-test/` (`SmtpTestView`) that displays the live SMTP config and sends a test message.
- `TIME_ZONE = "Asia/Jakarta"` for both Django and Celery (`USE_TZ=True`).

## Common commands

All via `Makefile` (uses `uv run`):

```sh
make dev        # runserver on :8000
make mmg        # makemigrations
make migrate    # migrate
make lint       # ruff format + ruff check --fix
make upgrade    # uv sync + uv lock --upgrade
make tw-run     # tailwind watch
make tw-build   # tailwind one-shot build
make web        # cd frontend && pnpm run dev
make worker     # celery worker (sets OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES for macOS fork safety)
make beat       # celery beat (DatabaseScheduler)
make audit      # cd frontend && pnpm audit fix
make dock       # docker compose down/build/up -d --env-file .env.docker (then tails logs)
```

`update.sh` is the prod deploy script: `git pull` → `docker compose build` → `up -d` → wait for postgres healthy → `manage.py migrate`. The Docker image runs `docker/backend-entrypoint.sh`, which branches on `ROLE` env (`web` runs `migrate` + `collectstatic` before exec'ing CMD; `worker` and `beat` skip those).

Direct Django (when Make target missing): `uv run manage.py <cmd>`.

Run a single test: `uv run manage.py test <app>.tests.<TestClass>.<test_method>` (e.g. `uv run manage.py test jobs.tests.JobModelTests.test_create`).

Job crawlers (management commands — one-shot ad hoc; recurring crawls run through the Celery pipeline below):

```sh
uv run manage.py crawl_indeed    "<listing-url>" [--max-pages N] [--limit N] [--sleep S] [--dry-run]
uv run manage.py crawl_jobstreet "<listing-url>" [--max-pages N] [--limit N] [--sleep S] [--dry-run]
```

Both upsert `Job` rows by `url` inside `transaction.atomic`. Defaults: `--max-pages 1`, `--limit 20`, `--sleep` from each scraper's `DEFAULT_SLEEP`.

## Architecture

Django project rooted at `core/` with four domain apps: `profiles`, `jobs`, `assessment`, `billing`. `core` is also installed as an app (holds `BaseModel` + `AppSetting`, the admin dashboard, the REST API surface, the Mayar payments adapter, Redis realtime, and Discord notifications). The `billing` app owns `Plan`, `Subscription`, `SubscriptionStatus`, and `effective_price` (Open-to-Work discount logic).

### Shared base — `core/models.py`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordanmaulana/cariinkerja.id](https://github.com/jordanmaulana/cariinkerja.id) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
