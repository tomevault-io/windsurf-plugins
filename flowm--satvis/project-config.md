---
trigger: always_on
description: git submodule update --init
---

# AGENTS.md

## Setup

```sh
git submodule update --init
pnpm install
```

A single `pnpm install` at the repository root installs dependencies for both
the SPA and the `worker/` package (a pnpm workspace). CI uses `pnpm ci`.

## Commands

| Task              | Command                                                               |
| ----------------- | --------------------------------------------------------------------- |
| Dev server        | `pnpm dev` (proxies `/api` → <https://satvis.space>)                  |
| Full-stack dev    | `pnpm dev:worker` + `SATVIS_API_PROXY=http://localhost:8080 pnpm dev` |
| Build             | `pnpm build`                                                          |
| Test (CI)         | `pnpm test` (frontend) and `pnpm --filter satvis-worker test`         |
| Lint (CI)         | `pnpm lint` (runs frontend and worker lint)                           |
| Lint fix          | `pnpm lint:fix` (runs frontend and worker fixes)                      |
| Type-check only   | `pnpm type-check`                                                     |
| Refresh static GP | `pnpm update-gp` (writes the gitignored `data/gp/` snapshot)          |
| Deploy            | `pnpm deploy` (builds frontend, then deploys worker)                  |

Worker-only scripts run via `pnpm --filter satvis-worker <script>`.

CI runs `lint`, then `test` (frontend + worker), then `build`.

## Architecture

- **Frontend**: Vue 3 + Vite + CesiumJS + Nuxt UI (Tailwind). Single-page app in `src/`.
- **Worker**: Cloudflare Worker backend in `worker/` — a workspace package (`satvis-worker`) with its own `package.json`, installed by the root `pnpm install`. Uses Wrangler for dev/deploy. Has its own `lint`, `type-check`, `test`, and `generate-types` scripts (run via `pnpm --filter satvis-worker <script>`).
- **Satellite data (GP element sets)**: fetched from CelesTrak as OMM JSON.
  - The worker refreshes each group into Workers KV via a cron trigger (every 3 h) and serves `/api/gp/<group>.json`, `/api/groups.json`, `/api/metadata.json`.
  - Groups are declarative: core groups in `worker/src/config/groups.core.json`, plugin groups in `data/custom/<plugin>/groups.json` (`sources`/`satellites`/`select`/`rename`/`include`/`extraRecordsFile`). `pnpm --filter satvis-worker generate-groups` merges them into the gitignored `worker/src/config/groups.generated.json`.
  - **Worker-less mode**: `pnpm update-gp` runs the same evaluator and writes a static snapshot into `data/gp/` (gitignored). The app probes `/api/groups.json` and falls back to that snapshot.
- **Data assets**: `data/` also contains Cesium assets (imagery, textures, stars) and 3D-model plugins under `data/custom/`. Copied into `dist/` at build time via `vite-plugin-static-copy`.
- Entrypoints: `index.html`, `embedded.html`, `test.html` (all configured as Vite MPA inputs).

## Key quirks

- **Cesium static assets**: Vite copies Cesium engine assets from `node_modules/@cesium/engine` and `@cesium/widgets` into `dist/cesium/`. The global `CESIUM_BASE_URL` is defined as `"./cesium"` in `vite.config.ts`.
- **Git submodules**: Required — `data/` content depends on them. Run `git submodule update --init` before first build.
- **Build globals**: `__BUILD_DATE__` and `__BUILD_SHA__` are injected via `vite.config.ts` `define`.
- **Path aliases**: `@/*` → `src/*` (in `tsconfig.json`).
- **Formatting**: `oxfmt` (config in `.oxfmtrc.json`): `printWidth: 180`, `sortImports`, and `sortPackageJson` enabled.
- **Linting**: `pnpm lint` runs frontend `oxlint`, `oxfmt --check`, and `vue-tsc`, then the worker's own lint script.
- **Env files**: `.env.development` / `.env.production` — only PostHog keys (`VITE_POSTHOG_*`). See `.env.example`.
- **PWA**: Service worker via `vite-plugin-pwa` with Workbox caching strategies.
- **TypeScript**: Strict mode, `noUnusedLocals`, `noUncheckedIndexedAccess`. Unused vars must be prefixed with `_`.
- **Vue conventions**: Component names in templates must use kebab-case.

## Deployment

`pnpm deploy` builds the frontend and deploys the worker. The worker needs a KV
namespace bound as `GP_KV` (see `worker/wrangler.jsonc`). After the first
deploy, KV is empty until a cron run fills it — either wait for the cron
(≤ 3 h) or force a fill now against the deployed KV:

```
cd worker
wrangler dev --remote --test-scheduled
curl "http://localhost:8080/__scheduled?cron=23+*%2F3+*+*+*"
```

### Migrating a private plugin from `sync.sh` to `groups.json`

Private plugins (e.g. the maintainer's untracked `data/custom/ot-tle/`) used to
be shell scripts that `grep`/`sed`-ed the bundled TLE files. Rewrite each as a
declarative `data/custom/<plugin>/groups.json` (untracked; same trust model as
before — never commit private plugin data):

- **`satellites`** (preferred for known, individually-named satellites): an
  array of per-satellite rows, each co-locating a satellite's NORAD id, its
  expected upstream name, and its display name so a rename's three facts live
  together instead of being scattered across `select.noradIds` and `rename`:

  ```json
  "satellites": [{ "noradId": 43556, "upstreamName": "LEMUR-2-EMBRIONOVIS", "name": "FOREST-2" }]
  ```

  A row matches by `noradId` when present (else by exact `upstreamName`), is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Flowm/satvis](https://github.com/Flowm/satvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
