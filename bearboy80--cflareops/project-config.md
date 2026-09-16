---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What this is

cloudflareOps — a self-hosted dashboard to manage **multiple Cloudflare accounts** (Zones, DNS, Workers, Pages, R2 storage, D1 databases, usage analytics) from one place. Astro 5 SSR + React 19 islands on Cloudflare Pages, D1 (SQLite) for storage, Cloudflare Access (Zero Trust) for auth. Bilingual UI (zh default / en).

## Commands

```bash
npm run dev              # Astro dev server on :4321, HMR, live D1 (better-sqlite3), .dev.vars loaded, Access bypassed
npm run build            # Production build
npm run preview          # wrangler pages dev ./dist — verify the real build against workerd
npm run deploy           # build + wrangler pages deploy
npm run typecheck        # astro check + tsc --noEmit  (run before every commit)
npm run check            # biome check --write .  — format + lint + organize imports (autofix)
npm run check:ci         # biome ci .  — non-writing gate CI runs (.github/workflows/ci.yml)
npm run format           # biome format --write .   (formatting only)
npm run lint             # biome lint .             (lint only, no writes)
npm run test             # vitest run
npm run db:migrate       # apply D1 migrations locally
npm run db:migrate:remote# apply D1 migrations to remote
```

Single test / watch:
```bash
npx vitest run tests/unit/cf-client.test.ts     # one file
npx vitest -t "verifyToken"                       # by test name
npx vitest                                        # watch mode
```

First-time local setup: `npm install`, `cp wrangler.toml.example wrangler.toml`, `cp .dev.vars.example .dev.vars`, generate a 64-hex `ENCRYPTION_KEY` (`node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`) into `.dev.vars`, then `npm run db:migrate`.

## Architecture

**Request pipeline** (`src/middleware.ts`, run as `sequence`):
1. `errorBoundary` — wraps everything; any uncaught exception under `/api/*` becomes structured JSON via `apiErrorResponse` (stable machine `code`, e.g. `config.dbBindingMissing`, `config.dbNotMigrated`). Page routes fall through to Astro's default handler.
2. `accessMiddleware` — verifies the `Cf-Access-Jwt-Assertion` JWT against the team's JWKS (cached per team domain) and sets `locals.userEmail`. In dev, `shouldBypassAuth` (`src/server/auth.ts`) lets `DEV_MODE=true` skip auth — **but only if `CF_ACCESS_*` are unset** (mutually exclusive guard rail; keep Access vars unset locally or you get a 403).
3. `localeMiddleware` — bilingual routing.

**Every API route starts with `appContext(locals)`** (`src/server/context.ts`) → returns `{ db, key, userEmail }`, throwing typed `ConfigError`s if `DB` binding or `ENCRYPTION_KEY` is missing/invalid. This is the single choke point that guarantees a request has a DB, a decryption key, and an authenticated user.

**Per-user data isolation is enforced in SQL, not middleware.** Every account/cache query filters by `owner_email = ?` (the `userEmail` from `appContext`). See `src/server/db/accounts.ts`. When adding any query that touches user data, you MUST scope it by `owner_email`. Resources not visible to the current user surface as `NotFoundError` → 404.

**Cloudflare API access goes through `CfClient` only** (`src/server/cf/client.ts`). Business code must never `fetch` Cloudflare directly. `CfClient` wraps the official `cloudflare` SDK and normalizes errors into `CfApiError` (status + messages). Two escape hatches inside the class for endpoints the SDK doesn't cover cleanly: `raw()`/`rawEnvelope()` (v4 REST envelope) and `graphql()` (Analytics API). The file is heavily commented with SDK gotchas (e.g. `scripts.settings` vs `scripts.scriptAndVersionSettings` map to *opposite* URL paths; `/pages/projects` rejects explicit `per_page`; workerd `fetch` needs `this` bound to `globalThis`). Read those comments before changing client methods.

**R2 has one sanctioned exception to the CfClient boundary.** Bucket/object/settings/usage calls go through `CfClient` like everything else, but object transfers are browser-direct via presigned S3 URLs built in `src/server/r2Presign.ts` — the **only** file allowed to reference `*.r2.cloudflarestorage.com`. S3 credentials are derived per request from the account token (accessKeyId = `verifyToken().id`, secret = SHA-256 hex of the token) and are never stored or logged. Adding `downloadFilename` appends a `response-content-disposition=attachment` query param **before signing** → true attachment download. Object preview is a hybrid channel: media kinds (image/pdf/video/audio) load presigned GET URLs directly in tags (tag loads are CORS-exempt, so bucket CORS config is not required); text/markdown go through `GET .../content`, which relays ≤1 MB via `CfClient.getR2ObjectContent` (over limit → 413 with code `objectTooLarge`); markdown renders only inside a sandbox iframe (`sandbox=""` + srcDoc — same anti-XSS pattern as `EmailPreview`). R2 SDK gotchas: objects list with `delimiter='/'` returns folder prefixes in `result_info.delimited` (undeclared in SDK types), not in the result array; object keys must be per-segment encoded via `encodeR2ObjectKey` (the SDK splices keys into URLs raw).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bearBoy80/cflareOps](https://github.com/bearBoy80/cflareOps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
