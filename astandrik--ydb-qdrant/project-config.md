---
trigger: always_on
description: **Project**: `ydb-qdrant` – Qdrant-compatible Node.js/TypeScript service and npm library that stores and searches embedding vectors in YDB using a global one-table layout (`qdrant_all_points`) with exact KNN search.
---

## Summary

**Project**: `ydb-qdrant` – Qdrant-compatible Node.js/TypeScript service and npm library that stores and searches embedding vectors in YDB using a global one-table layout (`qdrant_all_points`) with exact KNN search.

**Usage modes**:
- HTTP server exposing a minimal Qdrant-compatible REST API (`/collections`, `/collections/{collection}/points/*`).
- Programmatic Node.js client via `createYdbQdrantClient` that uses the same service/repository logic without running a separate HTTP server.

**Core features**: namespace isolation via `api-key` / `userUid`, global `qdrant_all_points` table, float vectors serialized to binary (`embedding`), exact search over YDB KNN functions.

## Tech stack and layout

- **Runtime**: Node.js 18+ (ESM, `type: "module"`). CI currently uses Node.js 22.
- **Languages & tools**: TypeScript, Express, Vitest, ESLint, k6, YDB SDK.
- **Key directories**:
  - `src/` – main implementation:
    - `config/env.ts` – loads environment, search mode, session pool settings.
    - `logging/` – pino logger.
    - `ydb/` – driver, helpers, schema creation (`qdr__collections`, `qdrant_all_points`).
    - `repositories/` – YDB access for collections and points (one-table layout).
    - `services/` – domain services (`CollectionService`, `PointsService`, errors, smoke test).
    - `routes/` – Express routers for collections and points.
    - `server.ts` – Express app and HTTP wiring; `index.ts` – bootstrap (ready check + server start).
    - `package/api.ts` – programmatic API exported by the npm package.
  - `test/` – Vitest unit and integration tests mirroring the above structure.
  - `docs/` – architecture, deployment, vector dimensions, CI/evaluation details.
  - `loadtest/` – k6 soak and stress tests for HTTP API.
  - `.github/workflows/` – CI workflows (build, tests, integration, recall, load, publish).

When you need to change behavior, prefer **services** + **repositories** as the main extension points; keep routes and ydb client thin.

## Bootstrap and environment

**Dependencies**:
- From the repo root, **always run** `npm ci` (in CI or fresh clones) or `npm install` (local iterative dev) before any build, test, or lint step.
- The project uses `package-lock.json`; prefer `npm ci` for reproducible installs.

**Required env for real YDB access** (HTTP server, integration tests, smoke tests, programmatic client):
- `YDB_QDRANT_ENDPOINT`, `YDB_QDRANT_DATABASE`.
- One of the credential envs: `YDB_SERVICE_ACCOUNT_KEY_FILE_CREDENTIALS`, `YDB_METADATA_CREDENTIALS`, `YDB_ACCESS_TOKEN_CREDENTIALS`, or `YDB_ANONYMOUS_CREDENTIALS` (dev only).

**Optional env** (common): `PORT`, `LOG_LEVEL`, YDB session pool settings.

For many **unit tests, lint, and typecheck**, a live YDB instance is not required because YDB is mocked; the **integration and recall tests** do require a reachable YDB per the docs.

## Build, run, lint, and typecheck

Run these from the repo root:

- **Build**: `npm run build` – compiles TypeScript to `dist/` via `tsc -p tsconfig.json`.
- **Typecheck**: `npm run typecheck` – type-only check (`tsc --noEmit`), useful in PRs.
- **Run HTTP server**:
  - Development: `npm run dev` (tsx watch on `src/index.ts`, default port 8080). Requires appropriate YDB env if you plan to hit vector APIs.
  - Production: `npm run build` then `npm start` (runs `dist/index.js` with source maps).
- **Lint**: `npm run lint` – ESLint over `src/**/*.ts` and `test/**/*.ts` (config in `eslint.config.mjs`).
- **Smoke test**: `npm run smoke` – builds then runs `dist/SmokeTest.js` against YDB using the programmatic client. Requires YDB env.

For any substantial change, aim for `npm run lint && npm test && npm run build` to pass before proposing a PR.

## Tests and evaluation

- **Unit tests** (fast, no real YDB):
  - `npm test` – runs Vitest over `test/**` excluding `test/integration/**`.
  - `npm run test:coverage` – same as above with coverage collection.
- **Integration tests** (real YDB):
  - `npm run test:integration` – runs end-to-end flows against YDB, including the one-table layout; requires `YDB_QDRANT_ENDPOINT`, `YDB_QDRANT_DATABASE`, and credentials.
- **Recall / F1 evaluation** (one-table search quality):
  - `npm run test:recall` – runs the recall benchmark suite for approximate and exact modes.
- **Load tests (k6)** – HTTP-level performance tests; usually run manually or in dedicated workflows:
  - `npm run load:soak` – long-running soak test.
  - `npm run load:stress` – stress test (ramp-up profile).

When you modify search logic, point serialization, or collection/points repositories, consider running at least integration tests and, when relevant, the recall suite.

## CI workflows

The main GitHub Actions workflows (referenced by README badges) are:
- `ci-build.yml` – install, build.
- `ci-tests.yml` – lint, typecheck, unit tests (with coverage).
- `ci-integration.yml` – integration tests against YDB.
- `ci-recall.yml` – recall and F1 benchmarks for the one-table layout.
- `ci-load-soak.yml` / `ci-load-stress.yml` – k6 load tests.
- `publish-ydb-qdrant.yml` – publishes the npm package on GitHub Release `published` events.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [astandrik/ydb-qdrant](https://github.com/astandrik/ydb-qdrant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
