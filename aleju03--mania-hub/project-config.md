---
trigger: always_on
description: Mania Tracker (mania-tracker.com) is an osu!mania community site with two cooperating parts:
---

# Repository Guidelines

Mania Tracker (mania-tracker.com) is an osu!mania community site with two cooperating parts:

- **Frontend** (`src/`): TanStack Start + Vite + React 19, SSR via Nitro (node-server preset, self-hosted on the VPS; the Vercel preset only builds when `process.env.VERCEL` is set, kept as a rollback target). File-based routes in `src/routes/`; `src/routeTree.gen.ts` is generated, do not hand-edit.
- **Live backend** (`live-backend/`, maintained separately and gitignored here, so it has its own commits/branches/pushes): always-on Node service that ingests osu! scores from the score feed (an osu! API recent-scores poller, with a legacy oSC Socket.IO source behind it), keeps durable SQLite projections, runs a DB-backed job queue, and streams updates to browsers over SSE (port 7227). It is the source of truth for live surfaces when `VITE_LIVE_BACKEND_URL` is set. Its own guide is `live-backend/AGENTS.md`; read it before working under that directory.

Countries are dynamic, not hardcoded: the backend keeps a `country_registry` with per-country status (cold -> warm -> active, can pause) and feature tier (`indexed` / `maps_warm` / `live` / `snipes`). Visiting a cold country can activate it (rate-limited). A synthetic `GLOBAL` scope aggregates all tracked countries. Default/home country is `CR`.

## Deep-dive docs

This file is the condensed guide. `docs/` holds the maintained reference; read the file matching the task before working on that area, do not read them all up front.

- `docs/backend.md`: live backend module map, ingest flow, job queue + job types, HTTP surface + rate buckets, replay video export, retention/storage/caching.
- `docs/features.md`: per-feature models - snipes, top plays, maps, farm helper, activity, dan estimates, chart analysis, profiles/rankings, goals, my data, uploaded replays, skins privacy.
- `docs/packs.md`: card pack economy, streak/blitz arcade games, GOAT poll.
- `docs/discord.md`: maniabot Discord bot, `/communities` server directory.
- `docs/admin.md`: admin surfaces - mascot overlay, todos, analytics, BBCode image audit, `bugs:pull` setup.
- `docs/frontend.md`: route/component map, live data flow (SSE client, cross-tab sharing), client state, on-device replay video export, OG images, BBCode editor.
- `docs/companella-integration.md`: the Companella score-import beta (`/companella`), off by default. `docs/companella-api.openapi.yaml`, `docs/companella-client-guide.md` and `docs/companella-operations.md` are its API spec, client contract and runbook.

Keep one-off audits, investigation notes, and capture reports in `local-notes/` (gitignored), not in `docs/`. Do not commit them.

## Structure

- `src/routes/`: file-based routes (`createFileRoute`); shared shell, auth, country context, theme bootstrap, and live-backend bootstrap in `src/routes/__root.tsx`. Search params are validated per route and drive country scope, pagination, and filters. API/proxy routes in `src/routes/api/` (`/api/sync` is the analytics capture proxy into the backend's in-house analytics store, not a live-data fallback); admin pages in `src/routes/admin/`. Files prefixed with `-` are tests, not routes; do not delete them as stale.
- `src/components/` (feature folders), `src/store.ts` (Zustand store), `src/lib/` (server/data utilities). `src/lib/osu.ts` is a facade over `src/lib/osu/` domain modules (rankings, beatmaps, replay, dan, users, pattern-analysis, plus shared support modules).
- `live-backend/src/`: `server.ts` (boot), `ingest/`, `osc/`, `jobs/` + `workers.ts`, `features/` (one module per surface), `http/`, `live/` (SSE), `discord/`, `dan/`, `replay-video/`; schema in `migrations/001_initial.sql`, later tables migrated in `db.ts`.
- `scripts/`: replay capture and validation, bug-report export, i18n generators, dev helpers in `scripts/dev/`. Static assets in `public/`.

## Commands

The user usually has dev servers running locally (frontend `3000`, live backend `7227`); do not start dev servers or run builds unprompted.

- Frontend: `npm run dev` / `npm run build` / `npm run test` / `npx tsc --noEmit`. Single test: `npx vitest run path/to/file.test.ts` (`-t "name"` for one case).
- Backend (inside `live-backend/`): `npm run dev` / `npm test` / `npx tsc --noEmit` / `npm run verify` (tests + build).
- Sync prod DB to local (dev PC only, overwrites the local DB, never run on the VPS): `npm run live-db:update` (fresh VPS snapshot) or `npm run live-db:sync-from-vps` (reuses the newest existing backup, `--dry-run` supported). `--with-analytics` also pulls the separate analytics DB; `--analytics-only` pulls just that one.
- Production bug reports: `npm run bugs:pull` exports open reports, reporter identities, threads, and images to a new gitignored `local-notes/bug-reports/` folder. Use `-- --status all`, `-- --id <id>`, or `-- --search "text"` to narrow/expand; read the printed `README.md` and report files, and inspect local image paths as needed. Read-only; does not mark reports seen. Setup in `docs/admin.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aleju03/mania-hub](https://github.com/aleju03/mania-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
