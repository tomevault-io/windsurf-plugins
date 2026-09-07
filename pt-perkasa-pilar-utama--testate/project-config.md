---
trigger: always_on
description: Git for your test database. Bun 1.4 monorepo: `apps/api` (Hono), `apps/web` (SolidJS 2 RC), `packages/shared` (valibot contract). Read `docs/technical-specs/_index.md` before changing architecture; cite the spec section in the commit body.
---

# Testate

Git for your test database. Bun 1.4 monorepo: `apps/api` (Hono), `apps/web` (SolidJS 2 RC), `packages/shared` (valibot contract). Read `docs/technical-specs/_index.md` before changing architecture; cite the spec section in the commit body.

## Commands

| Task                                  | Command                                                                                                                                                             |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Full gate (run before every hand-off) | `bun run complete-check`                                                                                                                                            |
| Tests                                 | `bun test` or `bun test apps/api/src/modules/states`                                                                                                                |
| Lint and format                       | `bun run lint`, `bun run fmt:fix`                                                                                                                                   |
| Dev servers                           | `bun run dev` (API :7378, Vite :7379 proxying `/api`)                                                                                                               |
| Smoke a running API                   | `bun run smoke`                                                                                                                                                     |
| Seed a dev instance with the demo     | `bun run seed:dev` after the reset and before `bun run dev` (starts its own API if none is up; needs the compose engines; admin keeps the `apps/api/.env` password) |
| Wipe the dev environment              | `bun run reset:dev --yes [--engines]` (stop `bun run dev` first; `--engines` also rebuilds the demo schema)                                                         |
| Browser end-to-end (Playwright)       | `bun run e2e` (its own ports 7478/7479, so it runs beside `bun run dev`)                                                                                            |
| Engine contract suites                | `bun run contract` (needs the compose engines; fails on a skip)                                                                                                     |
| Set the version everywhere            | `bun run bump-version <version>` (`--check` reports drift)                                                                                                          |

The gate is green today. Keep it green: a change that adds a lint error or a failing test is not done.

## Where things go

| Concern                                  | Place                                                       | Never                                                     |
| ---------------------------------------- | ----------------------------------------------------------- | --------------------------------------------------------- |
| Request schemas, response schemas, enums | `packages/shared/src`                                       | Define a shape twice; derive types with `v.InferOutput`   |
| HTTP routes                              | `modules/<name>/<name>.router.ts`                           | Logic in a router or in `modules/index.ts`                |
| Parsing, status codes, envelopes         | `modules/<name>/<name>.handler.ts` with `lib/http` helpers  | `c.json` by hand for errors; use `AppError` subclasses    |
| Business rules                           | `modules/<name>/<name>.service.ts` (`createXService(deps)`) | Reading `process.env`; `lib/config` is the only reader    |
| SQL                                      | `modules/<name>/<name>.repository.ts`                       | SQL in a service                                          |
| Typed mock data                          | `modules/<name>/<name>.mock.ts`                             | `as const` mocks; type them with the schema's output type |
| SPA data access                          | `features/<name>/<name>.model.ts` via `lib/api-client.ts`   | `fetch` anywhere else                                     |
| SPA state                                | `features/<name>/<name>.presenter.ts`                       | Logic in JSX                                              |
| SPA markup                               | `features/<name>/<name>.view.tsx` with `components/`        | A component importing `features/`                         |

Composition root: `apps/api/src/index.ts` wires services and handlers; `modules/index.ts` mounts routers. Both are wiring only.

## Rules the linter enforces

- Cyclomatic complexity 10 per function. Split; never `oxlint-disable complexity`.
- 300 lines per file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PT-Perkasa-Pilar-Utama/testate](https://github.com/PT-Perkasa-Pilar-Utama/testate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
