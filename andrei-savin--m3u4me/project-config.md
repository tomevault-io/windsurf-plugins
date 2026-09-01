---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

m3u4me — a self-hosted, single-user, local-network IPTV M3U playlist manager. Playlists/channels never leave the box it runs on. Express backend + Vite-built React 19 frontend, persisted to a single JSON file on disk. Per the README, the codebase is AI-generated with the maintainer (a non-developer) reviewing/steering — keep changes readable and avoid introducing patterns that need heavy explanation. There is no test suite in this repo.

## Commands

- `npm run dev` — `tsx server.ts`; runs Express with Vite in middleware mode (HMR dev server), on port 8080 (override with `PORT` env var).
- `npm run build` — `vite build` → `dist/`.
- `npm run start` — `node server.ts` in production mode; requires `dist/` to already exist (run `build` first). Serves `dist/` statically with SPA fallback.
- `npm run preview` — `vite preview`.
- `npm run lint` — `eslint . && tsc --noEmit`. This is also the typecheck command; there's no separate `typecheck` script.
- `npm run clean` — `rm -rf dist`.
- No test runner is configured.
- Production process management is PM2 via `ecosystem.config.cjs` (`pm2 start ecosystem.config.cjs`).

## Architecture

### One backend file, one JSON database

- `server.ts` is the entire backend — a single Express app with every route registered inline inside `startServer()`. No router modules, no ORM.
- Persistence is a single JSON file, `data/db.json` (gitignored, auto-created on boot with empty arrays). `readDb()`/`writeDb()` synchronously read/rewrite the *whole file* on every mutation — there are no transactions and no migration framework (one-off migrations like `migrateShortIds()` just run unconditionally at boot). This is adequate only because the app is single-user/local; don't add code that assumes concurrent writers.
- Auth secrets live in a separate gitignored file, `data/auth.json`.
- In dev (`NODE_ENV !== 'production'`), `server.ts` creates a Vite server in middleware mode and mounts it; in production it serves `dist/` statically with an `index.html` catch-all for client-side routing.

### Data model — duplicated by hand across backend and frontend

`server.ts` and `src/apiClient.ts` each declare their own copies of `Playlist`, `Channel`, `EpgSource`, `ChannelPoolSource`, etc. There's no shared types package — when changing a shape, update both files.

- **`Playlist`** — has a `shortId` (small incrementing integer used in the public `/[shortId]` and `/[shortId]/epg` URLs) and an `exportId` (UUID, legacy long-form export route kept for backwards compatibility).
- **`Channel`** — belongs to one playlist + one category string; `order` drives manual drag-reordering.
- **`EpgSource`** — an XMLTV URL (optionally gzip) or Xtream Codes credentials. Parsed programme/channel data is cached **in memory only** (`epgCache` Map keyed by source id) — it is never written to `db.json`, so it's rebuilt from scratch on every server restart via `refreshEpgSource()`, which runs for every stored source at boot and again on a 5-minute interval check against each source's `refreshIntervalHours`.
- **`ChannelPoolSource` / `ChannelPoolEntry` / `ChannelPoolChangeLog`** — a separate "bulk source" concept, distinct from playlists: an Xtream account, a playlist URL, or an uploaded file that you browse and cherry-pick channels from into an actual playlist. Entries *are* persisted to `db.json` (mirrored into an in-memory `channelPoolCache` for the running session). Each refresh diffs old vs. new entries by stream URL and appends an added/removed/renamed changelog entry, pruned to entries newer than 90 days.

### Auth is bespoke — and unrelated to `AuthContext`

- Real auth: PBKDF2-hashed password + a one-time-shown recovery key, both in `data/auth.json`. Login issues a random token held only in an in-memory `Set` (`activeSessions`) — sessions do not survive a server restart. The middleware mounted at `app.use('/api', ...)` gates every `/api/*` route except `publicPaths` (`/auth/status`, `/auth/login`, `/auth/recover`). If no password has ever been set, auth is a complete no-op.
- The frontend stores the token in `sessionStorage` (`src/apiClient.ts`: `getSessionToken`/`setSessionToken`) and routes every call through `authFetch()`, which attaches `Authorization: Bearer …` and fires a global `auth-expired` window event on a 401 (handled in `App.tsx` to re-lock the UI via `LockScreen`).
- `src/contexts/AuthContext.tsx` (`useAuth()`) is a **vestigial, unrelated stub** — it always returns a hardcoded dummy local user and has no connection to the password system above. Don't conflate the two when touching auth.
- The short playlist/EPG URLs (`GET /:shortId`, `GET /:shortId/epg`) are registered outside the `/api` prefix and are therefore never auth-gated — intentional, since IPTV players/EPG grabbers hitting these can't supply a bearer token.

### Frontend data flow: no query library — hand-rolled fetch + event bus

- `src/apiClient.ts` exports one `api` object holding every REST call, plus fetch-on-mount hooks (`usePlaylists`, `useChannels`, `useEpgSources`, `useChannelPoolSources`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrei-savin/m3u4me](https://github.com/andrei-savin/m3u4me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
