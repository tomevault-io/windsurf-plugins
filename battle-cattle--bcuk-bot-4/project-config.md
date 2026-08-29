---
trigger: always_on
description: npx tsc --noEmit && npm test
---

# BCUK Bot 4 — Claude Code Instructions

## ALWAYS: Before Committing

```bash
npx tsc --noEmit && npm test
```

---

## Dev

```bash
npm run dev   # ts-node src/index.ts
npm test      # Vitest
```

---

## Directory Structure

- **`src/index.ts`** — entry point (see Startup Sequence below).
- **`src/db.ts` + `src/db/`** — DB facade + one module per domain (`users.ts`, `guilds.ts`, `customCommands.ts`, `counters.ts`, `sfx.ts`, `streamMonitor.ts`, `eventSub.ts`, etc.), each with a co-located `*.test.ts`. Import from `src/db.ts` only (see Critical Invariants).
- **`src/discord/`** — Discord bot bootstrap, guild registry, voice presence.
- **`src/twitch/`** — Twitch chat bot + Helix API client, with `eventsub/`, `monitor/`, `pricing/`, `timers/` subfolders.
- **`src/commands/`** — command routing/cooldowns/handlers shared by both platforms.
- **`src/audio/`** — voice playback and SFX (see Voice adapter design decision below).
- **`src/web/`** — Express app (`server.ts`) + route/controller files under `web/routes/` (see Web Server below).
- **`src/shared/`** — cross-cutting utilities: `config.ts` (env vars), `logger.ts`, `crypto.ts`, `mutationQueue.ts`, `statusStore.ts`, `textTemplate.ts`.
- **`src/types/`** — Express `Request`/session type augmentation.
- **`src/test-utils/`** — shared test fixtures/mocks.

---

## Tech Stack

- **Runtime:** discord.js v14, tmi.js (Twitch chat), `mediaplex` (Opus), express v5 + `express-session`/`express-mysql-session`, `mysql2`, `helmet`, `winston`.
- **TypeScript:** `strict: true`, target ES2024, `moduleResolution: NodeNext`. No `engines` field in `package.json`.
- **ESLint** (flat config, type-aware): `no-floating-promises`, `no-misused-promises`, and `no-console` are all **errors**, not warnings — these affect how code must be written (await/void everything, use `logger` not `console.*`). `no-explicit-any` is a warning; relaxed for `*.test.ts`.
- **Test/build scripts:** `npm run build` (`tsc -p tsconfig.build.json`), `npm run lint`, `npm run check:circular` (madge).

---

## Startup Sequence (`src/index.ts`)

Boot order in `main()`: verify DB connectivity (ping, exit 1 on failure) → wire Twitch/EventSub runtime callbacks → `reloadGuildRegistry()` (must complete before Discord connects, exit 1 on failure) → start Discord bot → start Twitch bot → start web panel → start schedulers (counter/reward-pricing/timer) → start Twitch monitor + EventSub (fire-and-forget, catch-and-log only, doesn't crash the process).

- **`uncaughtException`/`unhandledRejection`** both log and `process.exit(1)` — deliberate: there's no process supervisor, so the app fails loudly instead of limping on with a corrupted state. Don't add a handler that swallows and continues.
- **`shutdown()`** runs on `SIGINT`/`SIGTERM`: stops schedulers/EventSub/monitor/bots, disconnects audio, closes the DB pool, then exits 0.

---

## Web Server (`src/web/server.ts`)

Middleware order: `helmet` (custom CSP) → EJS views → static → rate limiters (general/streamdeck/session) → body parsers → `trust proxy: 1` → session (`express-mysql-session`, backed by the `sessions` MySQL table) → `res.locals.user`/csrfToken.

Auth middleware (`src/web/middleware.ts`), applied in this order:
1. **`requireAuth`** — session check.
2. **`requireGuildContext`** — re-derives the user's access level from the DB for the active guild on every request. **Must run before any `requireAccessLevel` check** — skipping it leaves a stale or absent access level that could bypass a mod/manager/admin gate.
3. **`requireAccessLevel(level)`** factory → `requireMod`/`requireManager`/`requireAdmin`.
4. **`requireOwner`** — separate global super-admin check via `is_owner`, not part of the `AccessLevel` ladder.
5. **`authenticateBearerToken()`** factory → `requireApiKey` (Streamdeck) / `requireCompanionKey` (companion app), for API routes that bypass session auth entirely.

---

## Critical Invariants

- **`mediaplex` must be the first import in `src/index.ts`** — registers the Opus provider. Never reorder.
- **Import DB functions from `src/db.ts` only**, never `src/db/*` directly. The facade wraps some functions with cache-invalidation side effects (`upsertUser`, `updateTwitchBotEnabled`).
- **BIGINT columns are strings** (`bigNumberStrings: true` on the pool) — never coerce to `Number`. This protects against precision loss on values that can exceed `Number.MAX_SAFE_INTEGER`, like Discord snowflakes — it does not apply to a BIGINT result you can prove is bounded well within that range (e.g. `COUNT(*)` on a small admin table). If you do parse one of those back to a number, say so at the call site (why this particular value is bounded) — don't let it read like the same blind coercion the rule forbids elsewhere. See `getRowCount` in `src/db/utils.ts` for the pattern.
- **Blank Twitch names → `NULL`** — `user.twitch_name` has a unique index; empty strings collide.
- **`mutationQueue`** for concurrent-unsafe DB writes — user mutations serialise through it.
- **POST routes redirect to `?error=code`** on failure; GET reads it and passes to EJS. Never render errors from a POST handler.
- **`src/discord/discordUtils.ts`** has `isDiscordNotFoundError` and `tryDeleteDiscordMessage` — import, don't duplicate.

---

## Access Levels


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Battle-Cattle/BCUK-Bot-4](https://github.com/Battle-Cattle/BCUK-Bot-4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
