---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Personal app to monitor and (eventually) control the owner's 2025 VW ID. Buzz
by talking to VW's servers the way the stock myVW app does. It is an
independent re-implementation of VW's official **myVW** iOS app
(<https://apps.apple.com/us/app/myvw/id1481486650>) — it speaks the same VW
backend protocol (reverse-engineered from that app; see *VW protocol* below)
rather than reusing any of its code. Planned/agreed future work lives in
**TODO.md** — check it before designing anything new.

## Commands

```bash
pnpm test                 # typecheck + lint, all packages (run before committing)
pnpm --filter @vwapp/backend dev          # Worker on localhost:8787
pnpm --filter @vwapp/mobile start         # Expo (app expects Worker on :8787 unless EXPO_PUBLIC_API_URL set)
```

**Deployable by anyone (no owner values committed):** every account/identity
value comes from gitignored env files created from the `*.example` templates
(root `.env`, `backend/.dev.vars`, `app/.env`) — see README "Deploy your own
instance". `backend/wrangler.jsonc` pins **no** `account_id`; deploy targets your
account via `CLOUDFLARE_ACCOUNT_ID` in the environment. Apple Maps is optional
(see *Data freshness* / `isMapsConfigured`).

**Production Worker:** deployed as `vwapp-api` from `backend/` with
`pnpm --filter @vwapp/backend run deploy` (the `deploy` script sources
`backend/.env` for `CLOUDFLARE_ACCOUNT_ID` so wrangler targets your account) →
`vwapp-api.<your-subdomain>.workers.dev`. Prod shares dev's InstantDB app +
`CREDS_ENC_KEY` (secrets = `.dev.vars`, pushed via `wrangler secret bulk`), so
the cron polls VW for real every minute once deployed.

**Releasing the app — ALWAYS TestFlight, NEVER OTA:** never ship changes to the
owner's device with an EAS Update (OTA / `eas update`); every release goes
through a full native build + TestFlight submission (`pnpm --filter
@vwapp/mobile run publish:ios`). OTA is unreliable here (and can't deliver
native changes — new native modules like `expo-secure-store` only ship in a
build), so it is deliberately not used. The EAS Update tooling below stays only
for ad-hoc Expo Go dev testing, not for delivering releases.

**Publishing the app (EAS Update → Expo Go):** owner-specific Expo config is
env-driven via `app/app.config.ts` (which augments the generic `app.json`):
`EXPO_OWNER`, `EAS_PROJECT_ID` (→ `extra.eas.projectId` + the `updates.url`), and
`IOS_BUNDLE_IDENTIFIER`. Publish with `pnpm --filter @vwapp/mobile run update --
--message "..."` (the `run` is required — `pnpm update` is pnpm's built-in dep
updater) — the `update` script sources `app/.env` first because
`eas-cli`'s project resolver reads `EAS_PROJECT_ID` from the real shell env, not
from the dotenv-loaded config (`expo` does load it, `eas` does not). The API URL
is resolved at runtime in
`app/src/rpc.ts`: explicit `EXPO_PUBLIC_API_URL` (required for published builds)
→ else Metro dev-server host (port 8787; works for simulator and LAN devices —
`dev:worker` binds 0.0.0.0 for this) → else it throws (no hardcoded prod
fallback). Keep `runtimeVersion: {policy: "sdkVersion"}` — Expo Go can only load
updates whose runtime is its own SDK (`exposdk:56.0.0`).

Backend scripts (run from `backend/`, plain Node ≥24 runs the TS directly):

```bash
node --env-file=../.env --env-file=.dev.vars scripts/smoke.ts   # end-to-end vs running Worker, real VW login
node --env-file=.dev.vars scripts/auth-check.ts                 # auth plumbing only, no VW traffic
node --env-file=.dev.vars scripts/wipe.ts                       # delete ALL InstantDB rows + users
npm run status            # from packages/poc — direct VW status check, no Worker needed
```

InstantDB schema/perms changes: edit `packages/db/src/index.ts` (and
`backend/instant.perms.ts`), then from `backend/`:

```bash
set -a && source .dev.vars && set +a && npx instant-cli push schema --yes  # or: push perms
```

(instant-cli reads env vars, not `.dev.vars`, hence the sourcing.)

Crons never fire on a clock in dev; wrangler only exposes
`http://localhost:8787/cdn-cgi/handler/scheduled` (always on, no flag — the
older `--test-scheduled`/`__scheduled` is wrangler v3). `pnpm dev` therefore
runs `dev:worker` (wrangler) and `dev:cron` (a curl loop ticking that endpoint
every 60s) concurrently via pnpm's regex script runner; curl it manually for a
one-off tick.

## Architecture and the intent behind it

pnpm monorepo: `app/` (Expo SDK 56 + expo-router + Tamagui, runs in Expo Go),
`backend/` (Cloudflare Worker, oRPC), `packages/contract` (oRPC contract),
`packages/db` (shared InstantDB schema), `packages/poc` (throwaway reference
implementation of the VW protocol — relaxed lint, don't hold it to repo standards).

**Hybrid InstantDB design (deliberate):** the Worker's admin SDK is the *only
writer*; the app reads `vehicles`/`snapshots` directly via Instant live
queries (`db.useQuery`) and gets real-time updates — no client polling, no
cache invalidation for data. React Query is used *only* for RPC calls
(`login`/`logout`/`me`/`refresh`). `vwAccounts` (AES-GCM-sealed VW credentials

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sstur/vwapp](https://github.com/sstur/vwapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
