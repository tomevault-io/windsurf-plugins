---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

FlagshipRouter (`flagshiprouter-app`) — a local AI routing gateway + Next.js dashboard. It exposes one OpenAI-compatible endpoint (`/v1/*`) and routes traffic to **free** upstream providers with format translation, model-combo fallback, multi-account fallback, OAuth/API-key credential management, token refresh and usage tracking.

Two published artifacts live in this one repo:
- The **dashboard + gateway** (root `package.json`, `flagshiprouter-app`) — the Next.js server that does the actual routing.
- The **CLI launcher** (`cli/`, npm package `flagshiprouter`) — a separate package that starts the server, opens the browser UI and manages the tray. It has its own `package.json`, version, and build.

## Brand, provider policy and model renames — `brand.json` is the single source
- Never hardcode the product name, slug, data-dir name, CLI-tool provider key or repo URL. ESM code imports `BRAND` from `open-sse/config/brand.js`; CommonJS reads `cli/src/brand.js` (CLI) or `require("…/brand.json")` (src/mitm, updater). Locale files use `{{brand}}` / `{{slug}}` placeholders filled by `src/i18n/runtime.js`.
- `npm run brand:sync` (scripts/brand-sync.mjs) stamps brand.json into both package.json files; `cli/scripts/build-cli.js` runs it and ships a copy of brand.json.
- Free-only policy: `open-sse/providers/policy.js` (`isProviderAllowed`, `ALLOWED_REGISTRY`). Enforced in `src/shared/constants/providers.js` (dashboard + management APIs), `createProviderConnection` (connection store), `getProviderCredentials` and the chat handler (routing). The engine registry itself stays complete.
- Model renames: `open-sse/services/modelRenames.js`. `getModelInfo` resolves public ids to targets; `handleSingleModelChat` wraps the response with `presentResponseAsModel`; `/v1/models` and `/api/models/catalog` list renames and hide targets.
- Internal plumbing names: `x-fr-*` headers (custom-server ↔ app), `FLAGSHIPROUTER_*` env vars.
- UI: new shell in `src/shared/components/Sidebar.js` / `Header.js` / `BrandMark.js`, tokens in `src/app/globals.css`; the Models screen (`src/app/(dashboard)/dashboard/models/page.js`) replaces the provider list (`/dashboard/providers` redirects unless custom endpoints are enabled).
- Tests: `tests/setup/providerPolicy.js` disables the free-only policy for upstream engine tests; `tests/unit/brand-policy.test.js` covers the brand layer with the real policy.

The code lives in `src/` (Next.js app + dashboard/compat APIs), `open-sse/` (the provider-agnostic routing/translation engine), `cli/` (the launcher package), and `tests/`.

## Commands

One command from a fresh clone (install → build CLI bundle when stale → start CLI + browser UI):
```bash
npm run launch            # scripts/launch.mjs; CLI flags after --, e.g. npm run launch -- -p 20130
```

Dashboard/gateway (run from repo root):
```bash
cp .env.example .env
npm install
PORT=20128 NEXT_PUBLIC_BASE_URL=http://localhost:20128 npm run dev   # dev (webpack, port 20127 by default via next dev)
npm run build && PORT=20128 HOSTNAME=0.0.0.0 npm run start           # production
```
- Bun variants: `npm run dev:bun` / `build:bun` / `start:bun`.
- Default runtime port is **20128** (dashboard at `/dashboard`, API at `/v1`).
- Lint: `npx eslint .` (config `eslint.config.mjs`, extends `eslint-config-next`).

CLI package (`cli/`):
```bash
npm run cli:pack       # build + npm pack from root
cd cli && npm run dev  # nodemon watch
```

Tests (vitest, in `tests/`, an **independent** ESM package — not wired into root `npm test`):
```bash
npm install                             # ROOT deps first — tests import from src/ which needs `open`, `undici`, etc.
cd tests && npm install                 # then tests' own deps (vitest) → tests/node_modules (allowed by tests/.gitignore)
npx vitest run                          # all tests; auto-discovers tests/vitest.config.js
npx vitest run unit/capabilities.test.js   # single file (path relative to tests/)
```
> The committed `tests/package.json` `test` script hardcodes Unix paths (`NODE_PATH=/tmp/node_modules …`) — a shared-install workaround from upstream. On Windows (or anywhere), ignore it and use the `npx vitest` form above; `vitest.config.js` resolves the `open-sse`/`@/` aliases from the repo root regardless of where vitest lives.
>
> **The suite is NOT expected to be all-green on a plain checkout.** ~938 pass, ~64 fail. Judge regressions with `tests/__baseline__/verify-no-regression.mjs`, not a raw run. Expected red:
> - 26 catalogued in `tests/__baseline__/known-fails.txt` (rtk, oauth-cursor-auto-import, translator-request-normalization, …).
> - `unit/embeddings.cloud.test.js` imports `cloud/src/handlers/embeddings.js` — the `cloud/` worker dir is **not in this repo**, so it always fails here.
> - `unit/xai-oauth-service.test.js` times out (5s) when the xAI endpoint-discovery fetch isn't reachable/mocked.
> - `real/*.real.test.js` make live provider calls — need credentials, skip otherwise.
- `*.real.test.js` under `tests/translator/real/` make live provider calls — skip unless credentials are set.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theRizwan/FlagshipRouter](https://github.com/theRizwan/FlagshipRouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
