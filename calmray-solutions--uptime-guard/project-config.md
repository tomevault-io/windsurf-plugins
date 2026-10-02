---
trigger: always_on
description: > **This file is the shared context for any AI agent working in this repo.**
---

# CLAUDE.md - Agent Guide for Uptime Guard

> **This file is the shared context for any AI agent working in this repo.**
> **KEEP IT CURRENT: whenever you change architecture, add/remove a feature, change
> commands, deploy targets, conventions, or the data model - update the relevant
> section and the "Current Status" block in the same change. Treat a stale entry
> here as a bug. Do not record secrets (tokens, account IDs, real database IDs).**

Last reviewed: 2026-08-12

---

## What this is

Uptime Guard is a self-hosted uptime / certificate / cron monitor that runs entirely
on Cloudflare's free tier - **no server, no container, no external database**. A single
Worker serves both the API and the React dashboard; a cron trigger runs the checks; D1
(SQLite) stores everything.

Marketing angle (see README): "fully Cloudflare-hostable without any server," one-click Deploy button.

**Stack:** Cloudflare Workers (`fetch` + `scheduled`), D1, Wrangler 4, React 18 + Vite,
plain CSS (no Tailwind), TypeScript throughout. Auth is zero-dependency Web Crypto
(PBKDF2 password hash, HMAC session tokens, TOTP, Web Push VAPID).

## Repo layout

```
worker/            Cloudflare Worker (API + cron + static asset serving)
  src/index.ts     Main entry: routing, auth, cron scheduler, settings, public status
  src/checks.ts    Check execution + retry-burst confirmation; status up|down|cf_protected
  src/script.ts    Custom-script monitors: parser + runner for the step DSL
  src/tls.ts       Raw-socket TLS handshake (cert expiry)
  src/session.ts   HMAC session token create/verify (returns epoch for revocation)
  src/totp.ts      TOTP + base32 secret generation
  src/push.ts      Web Push (RFC 8291/8292)
  schema.sql       Full schema, CREATE ... IF NOT EXISTS (bundled + auto-applied)
  migrations/      Numbered historical migrations (schema.sql is the source of truth)
  wrangler.toml    Committed template (worker-dir manual deploy)
  wrangler.prod.toml / wrangler.demo.toml   gitignored, real infra IDs
dashboard/         React SPA (built into worker asset bundle)
  src/App.tsx      Routing (pushState/popstate), auth gate, setup gate, poll wiring
  src/components/  Overview, ServiceDetail, Settings, LoginGate, SetupGate, PublicStatus, ...
  src/lib/         api.ts, derive.ts, usePoll.ts, useStatusAlerts.ts, push.ts, sound.ts
wrangler.toml      ROOT config for the "Deploy to Cloudflare" button (auto-provisions D1)
docs/              README assets: logo.svg, uptime-guard-walkthrough.gif, screenshots/
                   (renaming the GIF is how you bust GitHub's camo image cache)
```

## Commands

Run from repo root:

- `npm run build:dashboard` - build the SPA into the worker asset bundle
- `npm run dev` - build dashboard, then `wrangler dev` on the worker
- `npm run deploy` - `scripts/deploy-banner.mjs`: runs `npx wrangler@4 deploy` (wrangler 4 is
  required for D1 auto-provisioning), streams its output, then prints the deployed dashboard URL
  in a large ASCII banner. Extra args pass through: `npm run deploy -- -c worker/wrangler.prod.toml`

Deploy a specific target from `worker/`: `npx wrangler deploy -c wrangler.prod.toml`
(or `-c wrangler.demo.toml`). Set `CI=1` for non-interactive.

## Data model (D1, 8 tables)

`projects` (public flag + public_slug) · `services` · `checks` · `incidents`
(last_reminder_at, reminder_level) · `push_subs` · `settings` (key/value, holds
session_secret + telegram config + `default_project_seeded`) · `daily_stats` (SLA rollups) · `login_attempts`
(rate limiting). Defined in `worker/schema.sql`; applied on first request / cron via
`ensureSchema`.

## Deploy targets

- **Production:** https://vigil.calmray.team (CalmRay infra, `wrangler.prod.toml`)
- **Demo:** https://vigil-demo.calmray.team - read-only, DEMO_MODE, password `demo`,
  seeded mock data (`worker/scripts/seed-demo.mjs`), used for README screenshots/GIF
- **One-click button:** root `wrangler.toml` - D1 binding OMITS `database_id` on purpose
  so Wrangler auto-provisions the database at deploy time (needs wrangler >= 4.45)
- Public repo: `CalmRay-Solutions/uptime-guard`

## Conventions

- **No em-dashes** anywhere (docs or UI text). Use `-`. The user is strict about this.
- **No `Co-Authored-By: Claude` trailer** in commits - user authorship only.
- Plain CSS with OKLCH tokens; inline SVG icons; no CSS framework.
- Keep secrets out of git: `.dev.vars`, `wrangler.{prod,demo,test}.toml`, and
  `wrangler.autotest.toml` are gitignored. Scan before every commit.
- `scrollbar-gutter: stable` is set on `html` - do not add `overflow-y: scroll` on
  `html`/`body` (they already have `height:100%`; that combo breaks page scrolling).

## Versioning / releases

Semver pre-releases, tagged `vX.Y.Z-<stage>.N` and published as GitHub pre-releases.
Keep the version in root, `dashboard/` and `worker/` `package.json` in sync with the tag.

Plan:
1. **Beta** (current): `1.0.0-beta.N` - bump N for each release while features are still landing.
2. **Release candidate:** `1.0.0-rc.N` once features are frozen; bug fixes only.
3. **Stable:** `v1.0.0` (normal GitHub release, not pre-release).
4. **After 1.0:** patch `1.0.1` for fixes, minor `1.1.0` for new features, major `2.0.0` for breaking changes.

Do not go back to `0.x`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CalmRay-Solutions/uptime-guard](https://github.com/CalmRay-Solutions/uptime-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
