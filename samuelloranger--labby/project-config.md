---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Labby is a self-hosted homelab dashboard: one Bun process serving a JSON API, a Svelte SPA, and an SSE stream. No auth (it runs behind a LAN/VPN reverse proxy by design — see README security section). Config and service credentials live in a small SQLite database (`config/labby.db`, `src/server/db.ts`), seeded on first run via built-in migrations and editable from the Manage Services page (the Database icon in the header). All service credentials stay server-side; the browser only ever sees sanitized data.

## Commands

```bash
bun install && (cd src/web && bun install)   # deps live in two places: root + src/web/
bun run dev                              # API + watch on :8080 (does NOT serve the web SPA)
cd src/web && bun run dev                    # Vite dev server with /api proxied to :8080
bun test                                 # run all tests
bun test src/server/config/schema.test.ts   # run a single test file
bun run build                            # vendor-icons → build web → bundle server to dist/
bun run start                            # run the bundled dist/index.js (needs build first)
```

For real frontend dev, run **both** `bun run dev` and `cd src/web && bun run dev` — the Vite server proxies `/api` to the Hono server. The Hono server only serves the built SPA from `src/web/dist`, so `bun run dev` alone has no UI until you `bun run build`.

`bun run build` requires `scripts/vendor-icons.ts` to fetch from the dashboard-icons CDN (network needed); the Dockerfile also fetches the Manrope woff2.

## Architecture

**Single source of poll truth → broadcast → SSE.** The browser never polls on a timer. Flow:

1. `src/server/sse/scheduler.ts` runs one `setInterval` per active channel at `config.refreshSeconds[channel]`. Each tick calls the matching integration and `hub.publish(channel, data)`.
2. `src/server/sse/hub.ts` (`BroadcastHub`) caches the latest payload per channel and fans it out to all subscribers.
3. `GET /api/stream` (in `app.ts`, via Hono `streamSSE`) replays the cached snapshot on connect (instant widget fill), then streams live updates as named SSE events; 15s `: ping` keepalive.
4. `src/web/src/lib/stores.ts` opens **one** `EventSource`, routes each named event to a per-widget Svelte store, and flips `streamConnected`/`stale` on drop (EventSource auto-reconnects).

**Channels** (`src/server/types.ts` `Channel`): `monitor`, `docker`, `downloads:qbittorrent`, `downloads:transmission`, `adguard`, `jellyfin`, `beszel`, `weather`. This union is the contract — SSE event names, store keys, scheduler `refreshers`, and `refreshChannel` all key off it. Adding a channel means touching all four.

**REST endpoints coexist with SSE.** `GET /api/*` endpoints are the initial-render fallback and are reused by the scheduler internally. After any `POST` action (container start/stop/restart, torrent pause/resume, adguard protection), `app.ts` calls `refreshChannel(...)` to force an immediate refresh + broadcast so the UI updates without waiting for the next tick.

**Config is DB-backed, validated, and never crashes.** `src/server/config/loader.ts` holds a module-level `ConfigState` (`{ok:true,config}` or `{ok:false,error}`) and notifies listeners via `onConfigChange`. The whole dashboard JSON is stored in the `settings` table under the `dashboard` key; `readConfigRaw` reads it, seeding from the default migration on first run (and persisting that default back into the DB). `saveThemeSettings` and `POST /api/settings` write to the DB and re-validate; call `reloadConfig()` to re-read. The scheduler subscribes and restarts its timers on every config change. `src/server/config/schema.ts` is the Zod schema (discriminated union on widget `type`) plus helpers: `collectMonitorSites` (dedupes by `checkUrl`), `collectDownloadClients`, `hasWidgetType`, `sanitizeDashboard`. Invalid config → error state, server keeps running and widgets show an error.

**Settings & credentials.** `src/server/db.ts` is a tiny `bun:sqlite` wrapper (`getSetting`/`setSetting`/`deleteSetting`/`getAllSettings`) over a `settings(key, value)` table at `config/labby.db` (override with `LABBY_DB_PATH`). Service env vars are rows in this table; `GET /api/settings` returns them (minus the `dashboard` row) and `POST /api/settings` is **full-replace** — it writes the posted keys and deletes any stored key the payload omits (so clearing/removing a field in the UI persists), preserving the `dashboard` row. Posted values are also pushed into `process.env` so integrations pick them up without a restart. **The container must run as the uid that owns the mounted `config/` dir, or writes fail with `SQLITE_READONLY`** (see README/compose: `user: "1000:1000"`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samuelloranger/labby](https://github.com/samuelloranger/labby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
