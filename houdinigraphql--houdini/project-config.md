---
trigger: always_on
description: **Schema location**: the canonical schema is the `create_schema` const in `packages/houdini/src/lib/database.ts` (node is the authority — it's what runs in production). `plugins/tests/schema.sql` is generated from it via `pnpm --filter houdini sync-schema` and embedded by the Go test harness; never edit the `.sql` by hand. A vitest (`src/lib/schema.test.ts`) fails if the two drift. No migration system — on a schema change the orchestration DB is rebuilt; it's version-stamped via `schema_version`
---

# Houdini — Claude Notes

## Database

**Schema location**: the canonical schema is the `create_schema` const in `packages/houdini/src/lib/database.ts` (node is the authority — it's what runs in production). `plugins/tests/schema.sql` is generated from it via `pnpm --filter houdini sync-schema` and embedded by the Go test harness; never edit the `.sql` by hand. A vitest (`src/lib/schema.test.ts`) fails if the two drift. No migration system — on a schema change the orchestration DB is rebuilt; it's version-stamped via `schema_version` / `PRAGMA user_version` (see `connect_db`), so persisted databases from older compilers are detected as stale and recreated.

**Dual SQLite backends**: `plugins/db_zombiezen.go` (native, `!wasip1`) and `plugins/db_ncruces.go` (WASI, `wasip1`). Both implement the `Conn`/`Stmt`/`Row` interfaces in `plugins/conn.go`. All DB code must go through the interface.

**FK indices**: SQLite does not auto-create indices on FK columns. Add an explicit `CREATE INDEX IF NOT EXISTS` in `create_schema` for any FK column that appears in a `WHERE` or `JOIN`.

**FK deferral**: FKs use `ON DELETE CASCADE`; deferral is achieved at the connection level via `PRAGMA defer_foreign_keys = ON` (set in `openDb` on the TS side and in the Go connection pragmas), so constraint checks happen at `COMMIT`, not per-statement. This is intentional; pipeline steps batch-insert rows that temporarily violate FK integrity within a transaction. The Go test pool enforces FKs exactly like production (`NewTestPool` runs the same connection pragmas). Deferral only reaches the end of the current transaction, so a reference whose target is created by a later pipeline step can never be an FK — those are plain TEXT columns checked by validation rules instead (e.g. `selection_directives.directive`). When a step wraps work in `db.Transaction`, always check the error from the commit closure: a deferred FK violation surfaces at COMMIT, and swallowing it silently drops every row in the transaction.

## Testing

**Heuristic**: Browser-verifiable changes require two tests: a Go table test asserting the generated artifact doesn't change, and a Playwright e2e test for the runtime behavior. Pure pipeline changes only need the table test.

| What changed | Test type |
|---|---|
| Go plugin logic only (extraction, validation, codegen) | `tests.RunTable` in `packages/<plugin>/plugin/` |
| Browser-visible behavior (mutations, pagination, cache) | `tests.RunTable` (artifact) + Playwright in `e2e/kit/` or `e2e/react/` (behavior) |
| TypeScript runtime or generated output shape | Vitest `.test.ts` next to source |

Canonical example: `packages/houdini-core/plugin/validate_test.go`. TypeScript test helpers: `testConfig()` / `testConfigFile()` in `packages/houdini/src/test/index.ts`.

**Updating a golden artifact**: the artifact table tests (e.g. `selection_*_test.go`) compare the whole generated artifact with `require.Equal`, so when a codegen change shifts the expected output, do NOT hand-edit the golden surgically — that's error-prone and has caused confusion. Instead: replace the entire expected value for that case with a placeholder (`tests.Dedent(\`PLACEHOLDER\`)`), run the test, copy the `actual:` string from the failure into place, then re-run and eyeball the diff to confirm the shape changed only the way you intended. Two tips: the failure prints `actual` already Go-escaped, so it can be pasted as a plain double-quoted string (no `Dedent`/backtick juggling needed); and when a change ripples across several cases, capture each `actual:` and splice it in (anchor on the case's unique hash if keys repeat) rather than editing by hand.

## React route typing

Per-route TypeScript typing (which `params` and `search` a route accepts, the `RouteHrefs` union, scalar resolution) lives in **one** place: `packages/houdini-react/runtime/routes.ts`. It derives everything from the generated manifest's shape via `typeof rawManifest`, and exports `RouteHrefs`, `ParamsForRoute<H>`, `SearchForRoute<H>`, `NavTarget<H>`, and `Goto`.

Anything that navigates to or references a route (`<Link>`, `goto`, `createMock`, and any future navigation/href API) must consume these shared types rather than re-deriving the rules or generating per-route type maps in Go. `formatMockFile` in `packages/houdini-react/plugin/runtime.go` is the example to follow: it imports the shared types and only generates its own mock-data types. URL construction (filling params, appending search, marshaling custom scalars) similarly goes through `buildHref` in `runtime/resolve-href.ts` — don't hand-roll it.

## Documentation

Docs live in `/docs` — framework-specific content under `/docs/svelte` and `/docs/react`, shared content (reference, extending-houdini, meta) under `/docs/shared`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HoudiniGraphql/houdini](https://github.com/HoudiniGraphql/houdini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
