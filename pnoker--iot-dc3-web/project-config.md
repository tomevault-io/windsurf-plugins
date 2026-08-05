---
trigger: always_on
description: Frontend for the IoT DC3 IoT platform (Vue 3 + Vite + Element Plus + Tauri).
---

# IoT DC3 Web

Frontend for the IoT DC3 IoT platform (Vue 3 + Vite + Element Plus + Tauri).

## Quick Reference

```bash
pnpm install                         # install deps (pnpm only — no npm/yarn)
pnpm dev                             # Vite dev server on port 8080
pnpm build                           # production build → dist/
pnpm check                           # vue-tsc type check
pnpm lint                            # eslint --fix + prettier --write
pnpm lint:check                      # eslint + prettier check only (no writes)

# Testing — Vitest
pnpm test                            # vitest run (all suites)
pnpm test:unit                       # vitest run tests/unit
pnpm test:api                        # vitest run tests/api
pnpm test:component                  # vitest run tests/component
pnpm test:views                      # vitest run tests/views
pnpm test:ci                         # vitest run --coverage (CI gate)
pnpm test:guard                      # vitest run tests/guardrails (AI coding guardrails)
pnpm test:coverage                   # vitest run --coverage

# Testing — Playwright E2E
pnpm test:e2e                        # playwright test (headless chromium)
pnpm test:e2e:headed                 # visible browser (E2E_HEADLESS=false)

# Full CI gate
make ci                              # lint-check + check + test-guard + test-ci + build
```

The backend API lives at `http://localhost:8000` (see `src/config/env/.env.dev`). The dev server proxies `/api` there
via Vite. **The dev server runs without the backend**, but login and data endpoints will fail.

## Package Manager

**pnpm** (pinned by `packageManager` field in `package.json` + Corepack).

- First-time setup: `corepack enable` once.
- **Do not use `npm` or `yarn`** — they will generate a second lockfile.
- When bumping pnpm, update the `corepack prepare pnpm@X --activate` line in `Dockerfile` in lockstep.

## Stack

| Concern      | Choice                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------- |
| Framework    | Vue 3.5 (`<script setup>`, Composition API)                                                 |
| Language     | TypeScript 6 (`verbatimModuleSyntax`, strict, noUncheckedIndexedAccess, noImplicitOverride) |
| Build        | Vite 8 + `@vitejs/plugin-vue` + `@vitejs/plugin-legacy`                                     |
| UI Library   | Element Plus 2.14 + `@element-plus/icons-vue`                                               |
| State        | Pinia 3 (setup-function style stores)                                                       |
| Routing      | Vue Router 5 (hash mode)                                                                    |
| HTTP         | Axios 1.16 + JSONBigInt (handles 64-bit integer IDs from Java backend)                      |
| i18n         | vue-i18n 11 (English + Chinese)                                                             |
| Charts       | @antv/g2, @antv/g6                                                                          |
| Maps         | @amap/amap-jsapi-loader                                                                     |
| Testing      | Vitest 4 (happy-dom) + Playwright 1.60 (E2E)                                                |
| Auto-imports | `unplugin-auto-import` + `unplugin-vue-components` (declarations in `src/config/ambient/`)  |

## Domain Model: Four-Layer IoT Entity

The core domain follows a strict hierarchy: **Driver** (protocol adapter) → **Profile** (device template) → **Device** (
physical equipment) → **Point** (data signal). This hierarchy is reflected everywhere — API paths, types, dashboard
palette colors, routing, entity enums.

## Project Layout

```
src/
├── api/                 REST API wrappers (thin, use crud* helpers from common.ts)
├── components/          shared components (cards, charts, tags, segmented, agentic)
├── composables/         usePagedList<T,Q> (generic paginated list), useEntityNames (ID→name cache)
├── config/
│   ├── ambient/         auto-generated type declarations (unplugin)
│   ├── axios/           axios instance + interceptors (auth headers, JSONBigInt, 401 redirect)
│   ├── constant/        enums, API base paths, auth header names, palette, icon map
│   ├── env/             dotenv files (NOT at repo root — Vite uses envDir: './src/config/env')
│   ├── i18n/            vue-i18n config + locale files
│   ├── plugins/         Element Plus + Highlight.js setup
│   ├── router/          routes + auth guards (common.ts, views.ts, settings.ts, operate.ts)
│   └── types/           all TypeScript interfaces (entity Form/Record pairs, dashboard, agentic)
├── store/               Pinia stores (auth, agentic, menu, interval)
├── styles/              Global SCSS
├── utils/               pure utility functions
└── views/               page-level components
```

## Key Conventions

### 1. `import type` is mandatory for type-only imports

`tsconfig.json` enables `verbatimModuleSyntax: true`. Any import used only as a type **must** use `import type` —
otherwise Vite keeps the import at runtime, the browser can't find the named export, and you get a `SyntaxError` → blank
page.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pnoker/iot-dc3-web](https://github.com/pnoker/iot-dc3-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
