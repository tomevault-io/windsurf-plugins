---
trigger: always_on
description: This file documents behavior expectations for dev branch schema support.
---

# Internal agent guidance

This file documents behavior expectations for dev branch schema support.
It is intended for contributors and automation working on this repo.

## Local testing

When changing code in `ts/`, validate in this order:

1. Run targeted package tests in `ts/` first.
2. If the change affects runtime behavior consumed by examples, rebuild `ts/dist`
   via `cd ts && npm run compile`.
3. Run only the relevant example tests while iterating.
4. Before landing broader runtime changes, run the full example matrix below.

Package-level commands:

- `cd ts && npm test -- --runInBand`
- `cd ts && npm test -- src/action/orchestrator.test.ts --runInBand`
- `cd ts && npm test -- src/action/transformed_orchestrator.test.ts --runInBand`
- `cd ts && npm run compile`

Codegen feature matrix:

- `go test ./internal/codegenmatrix -count=1`
- See `testdata/codegen_matrix/README.md` before adding schema/codegen options
  or generated-code regression coverage. New codegen inputs must be classified
  in `testdata/codegen_matrix/features.yml` as covered, skipped with a focused
  reason, delegated to an existing test, or non-codegen metadata. Prefer
  representative cross-feature interactions over exhaustive low-value
  combinations.
- The matrix includes SQLite and Postgres DB-codegen smoke fixtures. Postgres
  coverage runs when `DB_CONNECTION_STRING` or `ENT_CODEGEN_MATRIX_POSTGRES_URL`
  points at Postgres; otherwise that fixture skips in local runs. The Postgres
  smoke fixture runs both Node/pg and Bun/Bun SQL runtime variants.
- The matrix includes Bun runtime coverage via fixture runtime variants. Install
  Bun locally before running the full matrix, or use
  `ENT_CODEGEN_MATRIX_RUNTIME=node` while iterating on Node-only changes.
- Core DB-rendered features should declare
  `dialect_coverage: [sqlite, postgres]` in the matrix catalog so the test
  enforces coverage in both DB fixtures. Dialect-specific behavior should
  declare only the dialect it needs. Put shared core DB schema inputs in
  `testdata/codegen_matrix/fixtures/_shared/core_db` so both SQLite and
  Postgres fixtures exercise the same source.

Example prerequisites:

- Use Node 24 when validating the current example package locks and Docker
  images.
- Use Docker Compose v2 (`docker compose`). The example npm scripts call this
  form directly.
- Run `npm install` inside each example before validation.
- Use the package-local TypeScript binary because the examples do not expose a
  uniform `tsc` script: `./node_modules/.bin/tsc --noEmit`.

Full Ent example validation matrix:

- `examples/simple`
  `cd examples/simple && npm install && npm run rebuild-image && npm run codegen && ./node_modules/.bin/tsc --noEmit && npm test -- --runInBand`
- `examples/todo-sqlite`
  `cd examples/todo-sqlite && npm install && npm rebuild better-sqlite3 && npm run rebuild-image && npm run codegen && ./node_modules/.bin/tsc --noEmit && npm test -- --runInBand`
- `examples/ent-rsvp/backend`
  `cd examples/ent-rsvp/backend && npm install && npm run rebuild-image && npm run codegen && ./node_modules/.bin/tsc --noEmit && npm test -- --runInBand`
- `examples/ent-local-guide`
  `cd examples/ent-local-guide && npm install && npm run codegen && ./node_modules/.bin/tsc --noEmit && POSTGRES_TEST_DB=ent-local-guide POSTGRES_PORT=54329 npm test -- --runInBand`
- `examples/ent-semantic-notes`
  `cd examples/ent-semantic-notes && npm install && npm run codegen && ./node_modules/.bin/tsc --noEmit && POSTGRES_TEST_DB=ent_semantic_notes POSTGRES_PORT=54330 npm test -- --runInBand`

For `examples/todo-sqlite`, `npm rebuild better-sqlite3` keeps the native
SQLite module aligned with the active Node ABI. For the Postgres-backed local
examples, `npm run codegen` starts the DB container before generating code; run
`npm run db:down` in `examples/ent-local-guide` and
`examples/ent-semantic-notes` after validation to stop those containers. If a
fresh Postgres or pgvector container races readiness and tests fail with an
initial connection reset, rerun after the container is accepting connections.

`examples/ent-rsvp/ent-rsvp-web` is frontend-only and does not use Ent codegen
or the Ent Docker image; validate it with its own package scripts when changing
that app.

How examples pick up local `@snowtop/ent` changes:

- `examples/ent-local-guide` and `examples/ent-semantic-notes` already map
  `@snowtop/ent` to local `../../ts/src` in Jest, so their tests exercise local
  TS changes directly.
- `examples/simple` and `examples/todo-sqlite` still use the installed package
  by default. Their normal `npm test` runs do not prove local `ts/` changes.
- When validating local `ts/` changes against `simple` or `todo-sqlite`, use a
  temporary Jest override (or equivalent local-only config change) that maps:
  - `^@snowtop/ent$` -> `<rootDir>/../../ts/src/index.ts`
  - `^@snowtop/ent/(.*)$` -> `<rootDir>/../../ts/src/$1`
- Do not rely on a published package version for local validation of runtime
  fixes. If a temporary example-only test override is added for local
  verification, revert it before finishing unless updating the example test
  wiring is part of the intended change.

## Changelog

- Put unreleased changes in a top-level `## [Unreleased]` section above the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lolopinto/ent](https://github.com/lolopinto/ent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
