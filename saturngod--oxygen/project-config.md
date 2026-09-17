---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Oxygen — Video Transcoding & Live Streaming Platform

Laravel 13 + Inertia v3 + React 19 + Tailwind v4 web app plus **two independent Go services**:

- `golang-queue/` — VOD transcode worker. Consumes jobs from Redis, runs ffmpeg, writes HLS to S3.
- `golang-live/` — Live streaming service. Ingests RTMP, remuxes to live HLS (fMP4), tracks viewers. Controlled by Laravel via callbacks.

The Laravel app is the control plane (uploads, management, auth, session/viewer bookkeeping). Each Go service is its own Go module with its own `go.mod`, `.env.example`, and `CLAUDE.md`.

## Dev Environment

```bash
composer run setup          # first-time: install, migrate, build
composer run dev            # concurrently runs: php serve, queue:listen, pail, vite dev
composer run dev:octane     # same, but octane:start --watch instead of php serve
```

`composer run dev` does NOT start the webhook consumer, the scheduler, or either Go service. Start those separately when the feature under test needs them (see Background Processes).

If frontend changes aren't visible, the user likely needs `npm run build` or `composer run dev`.

Full local setup (Postgres/Redis/S3/env wiring) is in `README.md`; bare-metal Ubuntu production setup is in `Setup.md`. Static docs site: `docs/*.html`. Product specs live in `PRD/active/` and `PRD/backlog/`.

`AGENTS.md` is a symlink to this file. `GEMINI.md` is a separate, older copy — do not treat it as authoritative.

## Verification Pipeline

After changes, run the relevant steps:

```bash
vendor/bin/pint --dirty --format agent   # PHP formatting (always run after PHP edits)
npm run lint:check                        # ESLint (generated dirs are already ignored)
npm run format:check                      # Prettier on resources/
npm run types:check                       # TypeScript tsc --noEmit
php artisan test --compact                # Pest tests (SQLite in-memory, sync queue)
php artisan test --compact --filter=name  # single test
```

Full CI equivalent: `composer ci:check` (lint + format + types + tests).

## Generated Code — Do Not Hand-Edit

These directories are auto-generated and overwritten:

- `resources/js/actions/` — Wayfinder controller action functions
- `resources/js/routes/` — Wayfinder named-route functions
- `resources/js/wayfinder/` — Wayfinder internals
- `resources/js/components/ui/` — shadcn/ui components (use `npx shadcn` to update)

After adding or changing Laravel routes, run `php artisan wayfinder:generate` (or just `npm run dev` which triggers it via Vite).

Import Wayfinder functions from `@/actions/` (controllers) or `@/routes/` (named routes) — never hardcode URLs.

## Key Enums (Single Source of Truth)

- `App\Enums\VideoQuality` — 7 cases (Sd240p–Uhd2160p) with width/height/bitrate. **The frontend, the Go transcode worker, AND the Go live service must all mirror this enum.** If you change it, update all four in the same PR (PHP enum, `resources/js`, `golang-queue/internal/quality`, `golang-live`).
- `App\Enums\MediaFileStatus` — `uploaded`, `progress`, `success`, `failed`.
- `App\Enums\OrganizationRole` — `admin`, `operator`.
- `App\Enums\LiveStreamStatus` — `idle`, `live`, `offline`, `restarting`, `failed`, `disabled` (has `label()`).
- `App\Enums\LiveStreamSessionStatus` — `starting`, `live`, `ended`, `failed`.
- `App\Enums\WebhookEvent` — `file_uploaded`, `file_status_changed` (has `label()`).

## Models

`User`, `Organization` (belongsToMany via `organization_user` pivot with `role`), `Folder`, `MediaFile`, `MediaFileProfile`, `Profile`, `Webhook`.

Live streaming: `LiveStream` (owns stream, encrypted `stream_key`, status, restart flags), `LiveStreamSession` (one broadcast session: status, hls_url, viewer/segment metrics), `LiveStreamViewerRollup` (minute-level viewer snapshots).

## Organizations & Multi-Tenancy

Multi-tenant at the organization level. A user may belong to many orgs; the pivot carries `role` (`admin`|`operator`). At registration, a new org is created and the user attached as `admin`.

### Active organization (session-scoped)

Resolved in `HandleInertiaRequests::share()`:

1. Read `current_organization_id` from session.
2. Find that org in the user's memberships; fall back to first membership (alphabetical).
3. Rewrite session if stale.

Shared Inertia props: `auth.user.current_role`, `auth.user.current_organization`, `auth.organizations`.

When adding server-side code needing the active org, use `session('current_organization_id')` — do NOT re-derive from "first membership."

### Route permission layering (`routes/web.php`)

Three concentric groups — pick the right one, do not duplicate checks in controllers:

1. **`auth` + `verified`** — only `PUT organizations/{organization}/switch` sits here directly.
2. **`EnsureOrganizationMember`** (nested in 1) — normal app surface: dashboard, manage/*, status.
3. **`EnsureOrganizationAdmin`** (nested in 1) — `admin/organizations/{organization}/*`: settings, users, profiles, webhooks, **live-streams** (list/create/show/update + `rotate-key`/`restart`/`disable`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saturngod/oxygen](https://github.com/saturngod/oxygen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
