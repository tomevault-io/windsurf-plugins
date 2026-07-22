---
trigger: always_on
description: handles this in `dev` / `build` / `test`).
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Crowi is a Markdown-based Wiki application for team knowledge sharing. The
codebase is mid-migration from a legacy Express + Swig + jQuery monolith to a
modern Hono API plus a Next.js 16 (App Router) frontend.

**Current Status (Crowi 2.0)**: Turborepo monorepo with two apps and one
shared contract package. Most core wiki features (page CRUD / list / portal /
revisions / bookmarks / likes / seen-by / comments / watch / trash / backlinks
/ notifications / user pages / page history) have been migrated to the new
stack. Admin section is in progress (foundations + crypto migration UI +
security settings landed). See `TODO.md` for the up-to-date phase status.

## Monorepo Structure

```
crowi/
├── apps/crowi-runner/        # reference runner project (@crowi/runner-app): dev
│                             #   launch point + full Docker image build source.
│                             #   Owns @crowi/api + full plugin set + crowi.config.json.
├── apps/crowi-site/          # crowi.wiki LP + docs (Next.js + Fumadocs, :4303)
├── .env(.example)            # dev runtime env at repo root. dev loads it via the
│                             #   api `dev` script's `--env-file-if-exists=../../.env`
│                             #   (cwd is apps/crowi-runner); prod/operators rely on
│                             #   app.ts `dotenv.config()` reading the cwd's .env.
└── packages/
    ├── api/                  # Hono API (:4301)
    ├── api-contract/         # Hono (@hono/zod-openapi) contracts + Zod schemas
    ├── web/                  # Next.js 16 App Router (:4302)
    ├── runner/               # config loader + plugin resolver (used by api boot)
    ├── tsconfig/             # shared library/app-node/app-web tsconfig presets
    ├── admin-cli/            # `crowi-admin` CLI
    └── plugin-{api,aws,storage-*,renderer-*,search-elasticsearch}/
```

`ls packages/` and `tree -L 2 packages/api/src` give the rest. Highlights
worth knowing without reading code:

- `packages/api/src/hono/` — the Hono app: `handlers/` (endpoint handlers; `handlers/admin/` for admin), `middleware/`, `app.ts`
- `packages/api/src/crowi/index.ts` — boot sequence (encryption → DB → config → plugins → server)
- Plugin resolution: `@crowi/runner` reads `crowi.config.json` + uses
  `createRequire(<projectDir>/package.json)` to load plugin npm packages from
  the runner's `node_modules/`. Operators add a plugin by declaring it in
  their runner's deps + listing in `crowi.config.json:plugins`; the api never
  needs to be rebuilt. Dev path: `projectDir` = `apps/crowi-runner` (the
  monorepo's reference runner project `@crowi/runner-app`, which owns
  `@crowi/api` + the full plugin set + `crowi.config.json`); the api `dev`
  script `cd`s there before `tsx watch`, so dev resolves plugins exactly the
  way prod does. The full Docker image (`packages/api/Dockerfile`) is built by
  `pnpm deploy --filter=@crowi/runner-app`. `@crowi/api` itself is plugin-free
  (SDK `@crowi/plugin-api` + core only). `.npmrc` still hoists `@crowi/plugin-*`
  to the repo root for now.

## Tech Stack

- **API**: Hono 4 + `@hono/zod-openapi` + Mongoose + JWT auth (`jwtAuth` middleware)
- **Web**: Next.js 16 (App Router, Turbopack) + React 19 + Tailwind CSS v4 + shadcn/ui + @tanstack/react-query
- **Shared**: TypeScript 5.x strict, pnpm workspaces, Turborepo
- **Format / Lint**: Biome (format) + ESLint (lint), lefthook hooks
- **Tests**: Jest + supertest + mongodb-memory-server (API only; web tests TBD)

## Development Commands

Scripts live in root + per-package `package.json`. `pnpm <script>` filters with
`turbo` automatically. Non-obvious points:

- **Dev**: `docker compose up -d` for infra (mongo/redis/es/plantuml) →
  `pnpm dev` for api+web+plugins. `pnpm dev:api` / `pnpm dev:web` for one side.
- **Test infra**: `docker compose up -d` also starts `crowi-test-mongodb` (port
  27018, tmpfs data dir), a MongoDB dedicated to `@crowi/api`'s jest suite and
  kept independent from the always-on dev `mongodb` (27017) — a full parallel
  run's per-file create-db/autoIndex/dropDatabase churn no longer competes
  with dev data for the same disk path. tmpfs means it is fully disposable:
  a restart or `docker compose down` loses all data with no consequence
  (tests only ever write per-file scratch databases their own teardown
  already drops). `packages/api/src/test/global-setup.js` probes 27018 first,
  falls back to the dev `mongodb` (27017), and finally to an in-process
  `mongodb-memory-server` when neither is reachable — a checkout that hasn't
  pulled this compose change keeps working unchanged.
- **Targeted run**: `pnpm --filter @crowi/api <script>` to run a script in
  one package only.
- **Lint must be errors=0** (warnings tolerated). pre-push lefthook enforces.
- **Format**: Biome auto-runs on staged files (lefthook pre-commit). `pnpm
  format` only when bypassing hooks.
- **api-contract**: edit contracts/schemas → `pnpm --filter @crowi/api-contract
  build` to regenerate dts before api/web consumers pick them up (turbo `^build`
  handles this in `dev` / `build` / `test`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crowi/crowi](https://github.com/crowi/crowi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
