---
trigger: always_on
description: - Surge Web Dashboard (formerly YASD) is a React 19 single-page app for controlling Surge instances through the Surge HTTP API.
---

# AGENTS.md

## Project overview

- Surge Web Dashboard (formerly YASD) is a React 19 single-page app for controlling Surge instances through the Surge HTTP API.
- The app authenticates against a selected Surge host, then sends API traffic to that host's `/v1` endpoints.
- Remembered profiles and language/theme preferences are stored locally in the browser.

## Tooling and environment

- Node `24` is the expected runtime (`.node-version` is `24`, `package.json` requires `>=24 <25`).
- Package manager: `pnpm` (`packageManager: pnpm@10.32.1`).
- Vite Plus (`vp`) powers the dev server, linting, formatting, and tests.
- Path alias: `@/*` maps to `src/*`.
- TypeScript is strict and uses Emotion's JSX runtime (`jsxImportSource: @emotion/react`).

## Common commands

```bash
pnpm start                     # dev server
pnpm start:profile             # dev server with profiling enabled
pnpm lint                      # type-aware lint
pnpm lint:fix                  # auto-fix lint issues
pnpm test:types                # TypeScript typecheck
pnpm test                      # run Vitest once
pnpm test -- src/path/to.spec.ts
pnpm test:watch                # watch mode
pnpm test:coverage             # coverage report
pnpm verify-translation        # ensure zh keys match en baseline
pnpm build                     # Vercel-style production build with service worker
pnpm build:release             # CI release build + tarball bundle
pnpm build:surge               # Surge built-in bundle under /web
```

## Architecture

### App shell and routing

- `src/index.tsx` is the browser entrypoint: it loads global styles, initializes i18n, renders the router, and conditionally registers the service worker.
- `src/router/router.tsx` wraps the route tree with `AppContainer` and `App`, then creates either a browser router or hash router based on `VITE_HASH_ROUTER`.
- `src/routes.tsx` is the route map. Most feature pages are lazy-loaded; `routeOptions` control fullscreen pages and safe-area behavior.
- `AppContainer` wires Redux, Helmet, theme state, UI confirmation dialogs, safe-area support, and bootstrap-time initialization.
- `App.tsx` owns app-level SWR config, the page layout shell, traffic polling hookup, network failure modal, and version update checks.

### State and data flow

- Redux in `src/store` is intentionally small:
  - `history`: remembered Surge profiles from local storage
  - `profile`: the currently selected Surge target
  - `traffic`: rolling traffic stats used by the dashboard charts
- `profileActions.update` is the key handoff point: it calls `setServer(...)` in `src/utils/fetcher.ts`, which reconfigures the shared Axios client with the selected host, port, TLS mode, and `x-key` header.
- Most server reads use SWR on top of that shared Axios client (`src/utils/fetcher.ts`, `src/data/api.ts`, plus page-local hooks).
- `src/bootstrap/Bootstrap.tsx` restores remembered profile history and last-used language before rendering the app.

### Feature layout

- `src/pages/*` contains route-level screens: Landing, Home, Policies, Requests, Traffic, Modules, Scripting, DNS, Devices, and Profiles.
- `src/components/*` contains shared UI, providers, and cross-page building blocks.
- `src/components/ui/*` holds shadcn/ui-style primitives.
- `src/types/index.ts` is the central place for API response and domain types.

### Styling and UI stack

- The codebase uses a hybrid styling approach:
  - Tailwind CSS v4 + shadcn tokens in `src/styles/shadcn.css`
  - global CSS in `src/styles/global.css`
  - Emotion `css`/`styled` in many page and shared components
- When editing UI, match the surrounding style instead of forcing a single styling system.

### i18n

- i18n lives under `src/i18n` and lazy-loads locale JSON files.
- English (`src/i18n/en/translation.json`) is the baseline; `scripts/verify-translations.mjs` checks that `zh` contains every key from `en`.
- If you add or rename translation keys, run `pnpm verify-translation`.

### Build and runtime modes

- `scripts/build.mjs` drives release builds and post-processing.
- Important env toggles used by the app/build:
  - `VITE_HASH_ROUTER`
  - `VITE_RUN_IN_SURGE`
  - `VITE_URL_PATH_PREFIX`
  - `VITE_USE_SW`
  - `VITE_PROFILE`
  - local-dev connection helpers: `VITE_PROTOCOL`, `VITE_HOST`, `VITE_PORT`
- `pnpm build` targets the standalone/Vercel deployment flow.
- `pnpm build:release` and `pnpm build:surge` switch to hash-routing and bundle artifacts for release distribution.

## CI and commit conventions

- GitHub Actions runs on Node 24 and currently executes `pnpm build`, `pnpm verify-translation`, and `pnpm test`.
- Commit messages follow Angular-style commitlint rules with a 72-character header limit.

<!--VITE PLUS START-->

# Using Vite+, the Unified Toolchain for the Web

This project is using Vite+, a unified toolchain built on top of Vite, Rolldown, Vitest, tsdown, Oxlint, Oxfmt, and Vite Task. Vite+ wraps runtime management, package management, and frontend tooling in a single global CLI called `vp`. Vite+ is distinct from Vite, but it invokes Vite through `vp dev` and `vp build`.

## Vite+ Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geekdada/yasd](https://github.com/geekdada/yasd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
