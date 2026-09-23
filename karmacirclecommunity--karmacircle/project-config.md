---
trigger: always_on
description: - This is a monorepo: `apps/web` (frontend, `karmacircle-frontend`) and `apps/api` (backend, `karmacircle-api`). Read [docs/specs/README.md](./docs/specs/README.md) for the frontend's master map, and [apps/api/docs/specs/README.md](./apps/api/docs/specs/README.md) for the backend's — each covers only its own app.
---

## agentic workflow

- This is a monorepo: `apps/web` (frontend, `karmacircle-frontend`) and `apps/api` (backend, `karmacircle-api`). Read [docs/specs/README.md](./docs/specs/README.md) for the frontend's master map, and [apps/api/docs/specs/README.md](./apps/api/docs/specs/README.md) for the backend's — each covers only its own app.
- Read [docs/specs/known-issues.md](./docs/specs/known-issues.md) (frontend) and/or [apps/api/docs/specs/known-issues.md](./apps/api/docs/specs/known-issues.md) (backend) before touching any area either flags — duplicated implementations, dead code, unrouted pages, and validation that doesn't actually block submission are all cataloged there so you don't rediscover them the hard way.
- For anything touching a route path, method, or request/response shape, read [apps/api/docs/specs/api-contract.md](./apps/api/docs/specs/api-contract.md) first — it cross-references every backend route against exactly what the frontend calls, and documents where they currently disagree. Don't assume a route "just works" for the frontend without checking that file.
- There is no `PRODUCT_SPEC.md`, task-spec template, or Definition-of-Done doc in this repo yet — each app's `docs/specs/` is the closest thing to a source of truth today.
- There is one graphify knowledge graph at [graphify-out/](./graphify-out/), covering both apps' code (AST) plus `docs/specs/` and the top-level docs. Read `graphify-out/GRAPH_REPORT.md` before answering architecture questions — see the "graphify" section in `CLAUDE.md` for how to query and keep it updated.
- See "Testing — when, how, and where the data comes from" below before touching either app's test suite, or before claiming something is "tested."

## Testing — when, how, and where the data comes from

This repo is worked on by multiple AI agents (and humans), often in separate sessions with no shared memory of what a previous session already checked. This section exists so any agent — new to this repo or not — can answer "do I need to test this, and if so, how" without guessing. Read it once; don't rediscover it per session.

**What exists, in one sentence each:**
- `apps/api`: Jest + Supertest, every module covered, runs in-process against `mongodb-memory-server` — `npm test` / `pnpm --filter karmacircle-api test`.
- `apps/web`: Playwright E2E, organized feature-first under `apps/web/e2e/<feature>/` (mirroring `apps/web/src/features/<feature>/`), each folder with its own `README.md` — `pnpm test` (from `apps/web`) or `npx playwright test e2e/<folder>/` for one feature.
- Root `pnpm test` runs **both**, concurrently, via Turborepo — the API and the UI, at the same time, no server needs to be running beforehand. This is the one command to reach for by default.

**The full picture — read before writing or changing a test, not just before running one:**
[docs/specs/testing.md](./docs/specs/testing.md) (`apps/web`, and how the two apps' suites relate) and [apps/api/docs/specs/testing.md](./apps/api/docs/specs/testing.md) (`apps/api`). `docs/specs/testing.md` has a table pointing at every `apps/web/e2e/<feature>/README.md` — **check whether the feature you're touching already has one before writing new coverage**; it names what's covered, what's deliberately skipped (often because the underlying app code is unreachable/dead, not because testing it was skipped), and feature-specific test-data gotchas. A feature with no `e2e/` folder yet has no E2E coverage at all (see `docs/specs/testing.md#what-this-doesnt-cover`) — that's a gap to flag or fill, not to assume is covered elsewhere.

**Deciding whether to run tests for a given change:**
- Touched `apps/api/src/**`? Run `apps/api`'s Jest suite (`pnpm --filter karmacircle-api test`) at minimum. If you touched a module with no test file (check the coverage table in `apps/api/docs/specs/testing.md`), that's a gap — consider adding one rather than shipping the module still untested.
- Touched `apps/web/src/features/<name>/**`? Check whether `apps/web/e2e/<name>/` exists. If it does, run it (`npx playwright test e2e/<name>/`) and update it if the change affects behavior that folder's README says is covered. If it doesn't exist yet, the change is going out with zero automated coverage — say so plainly rather than implying it's tested.
- Touched something both sides depend on (a route contract, shared config, `app.ts`, `env.ts`) — run the full `pnpm test` from the repo root.
- Before claiming a bug is fixed or a feature works: reproduce it (or the feature's happy path) through the relevant test tooling or a manual check first — see CLAUDE.md's "start with reproducing the bug" rule. A claim of "tested" that wasn't actually run through anything is worse than saying it wasn't tested.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karmacircleCommunity/KarmaCircle](https://github.com/karmacircleCommunity/KarmaCircle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
