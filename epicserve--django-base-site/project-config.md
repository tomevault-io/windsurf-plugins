---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Django Base Site is an opinionated Django starter template with a production-ready foundation. The stack is Django 5 + django-ninja + django-allauth (headless mode, with MFA + WebAuthn passkeys) on the backend, and a Vue 3 SPA with Tailwind v4 on the frontend. Multi-tenant scaffolding (organizations, teams, invites), a notifications system (in-app + email, per-user category prefs, generic-target cleanup, retention purge), and an opt-in Stripe billing app (Checkout + Customer Portal, settings-declared plans + features, per-seat pricing, trials, coupons) are built in. Celery + Redis for background tasks, MinIO for S3-compatible local media storage, gunicorn in production.

## Architecture

- **Apps**: Under `apps/` — `accounts/` (custom user with timezone + avatar fields), `base/` (utilities, ninja error handlers, storage backends, the SPA shell view, the `qr_svg` view, etc.), `organizations/` (Organization, OrganizationMember, OrganizationInvite + the public invite ninja API), `teams/` (Team model + ninja API), `notifications/` (Notification + NotificationPreference, ninja API at `/api/notifications/`, `notify()` producer helper, retention purge via celery beat, GenericForeignKey post_delete cleanup driven by `settings.NOTIFICATIONS_TARGET_MODELS`, per-user category prefs declared in `settings.NOTIFICATIONS_CATEGORIES`), `billing/` (opt-in Stripe subscriptions tied to Organizations — see Billing section below).
- **Billing app** (`apps/billing/`, opt-in via `BILLING_ENABLED`): models are `BillingCustomer` + `Subscription` + `WebhookEvent`. Plans + features are declared in `settings.BILLING_PLANS` / `settings.BILLING_FEATURES` and loaded into `apps/billing/plans.py` and `apps/billing/features.py` dataclasses (mirrors the notifications-categories pattern). New subscriptions go through Stripe Checkout (full-page redirect); upgrades/cancels/payment-methods/invoices go through the Stripe Customer Portal. The webhook lives at `/webhooks/stripe/` (registered in `config/urls.py` outside the ninja API because it uses HMAC, not session+CSRF) and dedupes Stripe retries via `WebhookEvent`. Feature gating: `apps.billing.access.org_has_feature(org, key)` and the `requires_feature` decorator return `True`/pass through when `BILLING_ENABLED=False` so the starter template runs out of the box without Stripe credentials. Per-seat sync runs as a `transaction.on_commit` hook from `OrganizationMember` `post_save`/`post_delete`. Trial reminders + drift recovery run as celery beat tasks.
- **Settings**: `config/settings/` — `_base.py` (main, env-driven via epicenv), `__init__.py` (re-exports `_base`), `test_runner.py` (overrides for pytest), `e2e.py` (overrides for Playwright with pre-built Vite assets).
- **API**: A single `NinjaAPI` instance in `config/api.py` mounted at `/api/`. Routers come from each app's `api.py` (`apps.base.api`, `apps.accounts.api`, `apps.organizations.api`, `apps.teams.api`, `apps.notifications.api`).
- **URLs** (`config/urls.py`): A re_path catch-all serves the Vue SPA shell for every non-API path. `/_allauth/` mounts allauth's headless API, `/hijack/` mounts django-hijack, `/admin/` is the Django admin, `/api/` is the ninja API. A `_public_not_found` shim before the catch-all keeps stale `/public/static/*` chunks from being answered with HTML.
- **Frontend**: Vue 3 SPA in `frontend/` (was `src/` pre-conversion). `frontend/js/app.js` mounts `App.vue`, `frontend/js/router.js` defines all SPA routes (lazy-loaded). `frontend/js/stores/app.js` is the reactive app store; `appStore.fetchContext()` hits `/api/app-context/` to populate user, org, organizations, version, etc. `frontend/css/app.css` is Tailwind v4 with Fraunces / IBM Plex Sans / JetBrains Mono via Google Fonts. Built with bun + Vite.
- **Docker**: `compose.yml` with healthchecks on every service: `db` (postgres 17), `redis` (7), `mailpit`, `minio`, `web`, `worker` (celery), `frontend` (bun running Vite). The web container runs `migrate` and `ensure_s3_bucket` on startup. Multi-stage production image at `config/docker/Dockerfile.web` (python-requirements → base → dev / js_assets → prod with gunicorn).
- **Static / media**: WhiteNoise serves Vite-hashed assets in production with `Cache-Control: max-age=31536000, immutable` (regex defined in settings). Media uploads use `apps/base/storage.py:S3MediaStorage` which handles the Docker-internal vs. browser endpoint URL split for MinIO.

## Development Commands

Use Just for all development tasks. Common ones:

**Setup & Management:**
- `just init` - First-time setup: brings services up, runs `just create_superuser`, then attaches — use after `scripts/start_new_project` or after wiping the DB. Use `just start` for every subsequent boot.
- `just start` - `docker compose up`
- `just start_with_debugpy` - same with debugpy listening on `:5678`
- `just stop` - Stop all services
- `just build` - Rebuild Docker images + clear node_modules + collectstatic
- `just build_frontend` - `bun run build` + collectstatic
- `just clean` - Remove caches, coverage, dist
- `just create_env` - Generate `.env` from the schema in `.env.toml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [epicserve/django-base-site](https://github.com/epicserve/django-base-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
