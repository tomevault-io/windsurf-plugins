---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Full context lives in `AI_CONTEXT.md`** — it covers the collector pattern, RouterOS quirks, security invariants, and testing conventions in detail. Read it before making architectural decisions.

---

## Commands

```bash
# Rebuild and restart the container (do this after every source change)
docker compose build && docker compose up -d

# View live logs
docker logs -f mikrodash

# Run all tests (test/ is excluded from the image — copy first)
docker cp test/. mikrodash:/app/test
docker exec mikrodash node --test --test-force-exit /app/test/

# Run a single test file
docker exec mikrodash node --test /app/test/production-resilience-regressions.test.js

# Run locally without Docker (after npm install + node patch-routeros.js)
node src/index.js
```

---

## Architecture

MikroDash is a **single-process Node.js server** (no build step, plain CommonJS). The browser gets a static SPA; all live data flows over a single Socket.IO connection. There are no REST endpoints for live data — everything is pushed server→client.

```
RouterOS binary API (TCP)
        │
   src/routeros/client.js   ← ROS class: connectLoop, write(), stream()
        │
   src/collectors/          ← 15 domain collectors, orchestrated by index.js
        │                                        │
   Socket.IO emit            ← one named event   src/db-writer.js → src/db.js (SQLite)
        │                      per collector        time-series: traffic, ping, bandwidth
   public/app.js             ← ALL frontend logic in one file
```

**`src/index.js`** is the hub:
- `buildSession(routerCfg)` — creates ROS + all 15 collectors wired together
- `teardownSession(session)` — clean shutdown for hot-swap
- `sendInitialState(socket)` — replays `lastPayload` from every collector on new connect
- `connTableCache` — shared between `connections.js` and `bandwidth.js`
- All REST endpoints (settings, routers, dashboard layout, auth)

**Collectors** follow a strict contract: `start()`, `stop()`, `lastPayload`, `pollMs`, `state.last<n>Ts`, `state.last<n>Err`. See `AI_CONTEXT.md` → "Collector delivery model" for the streaming-vs-polling breakdown for each collector.

**Settings** are AES-256-GCM encrypted at `/data/settings.json` — managed by `src/settings.js` (`load`, `save`, `getPublic`, `isMasked`). Router configs live at `/data/routers.json` via `src/routers.js`; `activeRouterId` in settings points to the active entry.

**Database** (`src/db.js`) — SQLite via `better-sqlite3`, opened at `/data/mikrodash.db`. Schema is managed by numbered migrations in `MIGRATIONS[]`. Stores time-series data: `ping_samples`, `traffic_samples`, `bandwidth_usage`, `alert_events`, `connectivity_events`. `src/db-writer.js` is the write facade: it accumulates raw per-second traffic/bandwidth samples into 1-minute bucketed averages before flushing, so the DB never sees raw per-second rows. Call `db.open()` once at startup; `db.close()` on shutdown.

**Auth** — two layers that co-exist:
- Legacy HTTP Basic Auth (`src/auth/basicAuth.js`): enabled via `BASIC_AUTH_USER`/`BASIC_AUTH_PASS` env vars; covers all HTTP routes + Socket.IO engine.
- Session auth (`src/auth/sessionStore.js` + `src/users.js`): cookie-based (`mikrodash_sid`), users stored in `/data/users.json` with scrypt-hashed passwords, roles `admin`/`viewer`, optional `allowedRouterIds` per user. Login UI at `public/login.html` + `public/login.js`; `public/preflight.js` is the client-side auth gate loaded before `app.js`.

---

## Hard constraints

- **No build step.** CommonJS only — no TypeScript, no bundler, no transpiler.
- **No new runtime deps** without explicit approval. (`better-sqlite3` is approved and in use.)
- **Streaming-first.** Prefer `/listen` (event-driven) over `=interval=N` (timed push) over `setInterval` (polling). See `AI_CONTEXT.md` for the full rule.
- **No CDN.** All frontend assets live in `public/vendor/` (read-only — never modify).
- **`sanitizeErr(e)`** before any error reaches the browser. Never send raw `.message` or stack traces.
- **`esc()`** around every user-supplied string injected into HTML in `app.js`.
- **Credentials** are encrypted at rest. Always call `isMasked()` before writing a credential field on save.
- **User passwords** are scrypt-hashed (`src/users.js`). Never store or log plaintext passwords. `verifyPassword()` uses `crypto.timingSafeEqual` — don't replace it with a simple string compare.
- **Session tokens** are 32-byte random hex strings. Never expose them in logs, error messages, or API responses beyond the `Set-Cookie` header. Use `sessionStore.parseCookieHeader()` + `sessionStore.getSession()` to validate incoming requests.

---

## Versioning rule

**Do not bump `package.json` version or edit `CHANGELOG.md` during a working session.** Version bumps happen only when the user says "package it up" or equivalent. One bump covers the entire session.

---

## Testing

- Runner: `node --test` only — no Jest, Mocha, or other frameworks.
- Test the collector's output payload shape and values, not internal implementation details.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SecOps-7/MikroDash](https://github.com/SecOps-7/MikroDash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
