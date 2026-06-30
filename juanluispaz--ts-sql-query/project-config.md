---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

ts-sql-query is a type-safe SQL query builder (not an ORM) for TypeScript that supports MariaDB, MySQL, Oracle, PostgreSQL, SQLite, and SQL Server, running on Node.js (>=22) and Bun. Published documentation lives at https://ts-sql-query.readthedocs.io/ and is built from [docs/](docs/) with `mkdocs`.

## Commands

The project supports both Node and Bun. **Day-to-day development prefers Bun** (faster, native SQLite, no compile step for examples); the **publish pipeline and CI stay on npm/Node** — keep both paths working but reach for `bun` first when running things locally. Each script in `package.json` has a **single name** (no `bun:`-prefixed twin): the underlying shell script detects whether `bun run` or `npm run` invoked it (via `npm_config_user_agent`) and dispatches to `bun test`/`bun run` or `vitest`/`tsx` accordingly.

- **Typecheck only** (closest thing to a lint):
    - **For `src/`**: prefer `bun run validate:tsgo` (or `npm run validate:tsgo`) — runs `tsgo --noEmit` (TypeScript 7 Go-based compiler, ~6× faster). Fall back to `bun run validate` / `npm run validate` (`tsc --noEmit`) for the authoritative TS 6 check when diagnosing a discrepancy or before pushing to CI. `tsc` stays authoritative for src because the published typings have to compile in consumers' tsc — that constraint won't go away until tsgo is fully GA. CI runs both.
    - **For the new `test/` matrix**: **`tsgo` is authoritative** and `bun run validate:tests` / `npm run validate:tests` runs it against the full [`test/tsconfig.json`](test/tsconfig.json). Negative type tests follow tsgo's conventions for `@ts-expect-error` placement. The sub-experience is `bun run validate:tests:tsc` / `npm run validate:tests:tsc`, which runs `tsc` against [`test/tsconfig.tsc.json`](test/tsconfig.tsc.json) — same config, minus any file the two compilers can't agree on (the current exclude list lives in that tsconfig; see [`test/WRITING_TESTS.md`](test/WRITING_TESTS.md#handling-tsgo--tsc-divergences) for when to add to it). Both run in CI. The role split applies only to tests because tests don't ship — there's no downstream tsc-compat obligation, so the agent's default loop gets tsgo's speed and stricter span behaviour.
- **Build the publishable artifact**: prefer `bun run build:tsgo` (or `npm run build:tsgo`) for fast local sanity checks of the build output — wipes `dist/`, runs `tsgo -p tsconfig.build.json`. The publish flow (`npm run dist*`) still runs `npm run build` (`tsc -p tsconfig.build.json`) intentionally — `tsgo` is preview, so the artifact that actually ships is the one `tsc` emits. Don't switch the publish path to bun or tsgo.
- **First-time setup** (after clone or after pulling Prisma schema changes): `bun install && bun run generate-prisma` (or the `npm` equivalents). Examples that use Prisma will fail without this.
- **Run the full example matrix** (requires Docker, ~13 min): `bun run all-examples` (Bun-native) or `npm run all-examples` (Node/tsx). Same script name, runtime is detected. The Oracle driver now ships in thin mode, so no Oracle Instant Client install is required.
- **Run docker-free examples**: `bun run no-docker-examples` locally; `npm run no-docker-examples` is what CI runs. Both cover documentation examples plus SQLite/PgLite backends.
- **Run a single example/test**: `bun ./src/examples/<File>.ts` (or `tsx ./src/examples/<File>.ts`). Each example is a self-contained script with its own `main()`.
- **Coverage** (legacy `src/examples/` suite): `npm run coverage` (wraps `all-examples` with `nyc`; HTML report in `coverage/index.html`). Node-only; the script pins `npm_config_user_agent` so it always takes the tsx branch — don't invoke it via `bun run`. For the new `test/` matrix, use `bun run tests --coverage` (or `--use-vitest --coverage` for V8-rich coverage) — it lands under `.test-report/coverage/` and is documented in [`test/CLI.md` § Coverage](test/CLI.md#coverage).
- **Publish**: `npm run dist` / `dist-alpha` / `dist-beta` (Node only — pipeline expectation).
- **Docs preview**: `npm run docs` (requires `mkdocs` from the `.venv`).
- **The new `test/` matrix** (separate from the legacy `src/examples/` suite above): use the `tests:*` CLI family. Each has `--help`. Full guide split per topic — see [`test/README.md`](test/README.md) for the navigation map; rules live in [`test/DESIGN.md`](test/DESIGN.md), commands and flags in [`test/CLI.md`](test/CLI.md), wall-time numbers in [`test/BENCHMARKS.md`](test/BENCHMARKS.md).
    - `bun run tests` — full matrix, parallel, mocked (no docker, no real WASM). ~8 s for ~14k tests under bun (~60 s under vitest — see [`test/BENCHMARKS.md`](test/BENCHMARKS.md)). Cheap enough as a pre-push sweep; **not** the inner-iteration loop (see "Test-loop discipline" below — prefer `tests <coord>` or `--run-versions newest`).
    - `bun run tests --docker` — same matrix, docker-backed connectors hit their real DB (container reuse on by default). ~4:30 with warm containers — see [`test/BENCHMARKS.md` § Bun vs vitest](test/BENCHMARKS.md#bun-vs-vitest) for the runtime comparison.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juanluispaz/ts-sql-query](https://github.com/juanluispaz/ts-sql-query) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
