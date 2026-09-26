---
trigger: always_on
description: Read-only reconnaissance against `main` @ 29ea0099 (v1.45.1). Everything an
---

# Recon: project conventions and the gates that fail a pull request late

Read-only reconnaissance against `main` @ 29ea0099 (v1.45.1). Everything an
implementer on this release must know before writing a line.

## Tests

All tests live under `tests/`, none co-located in `src/` (1018 files). Top-level
directories mirror the `src/` layers. Two naming styles coexist: the current
directory form `tests/core/brain/<module>.test.ts`, and a legacy dot-flattened
form in `tests/core/` only. New files use the directory form. Python is
`tests/python/test_*.py`.

The dominant vault fixture (186 files) is not the helper: it is
`mkdtempSync(join(tmpdir(), "o2b-<suite>-"))` plus `bootstrapBrain(vault)` with
`rmSync` in `afterEach`. `tests/helpers/fixtures.ts` exists but only four files use
it. Helpers cover cross-cutting machinery instead: `run-cli.ts`,
`search-fixtures.ts`, `mock-embedding.ts`, `sqlite-vec.ts`, `fake-http.ts`.

`tests/setup.ts` is preloaded from `bunfig.toml` and guarantees two things: a
pinned `O2B_DEVICE_ID`, and a hermetic throwaway config plus vault when
`OPEN_SECOND_BRAIN_CONFIG` is unset. `bootstrapBrain` requires the default config
path to exist, which is exactly what the preload provides.

Two hazards. First, never delete `OPEN_SECOND_BRAIN_CONFIG`, `XDG_CONFIG_HOME`,
`O2B_DEVICE_ID` or `VAULT_DIR` without restoring them: `bun test` shares one
process, and a test that deletes without restoring strips the hermetic default
from every file ordered after it. That shipped, and 1.45.1 fixed it. The
save-and-restore idiom is `tests/core/config-read-failure.test.ts:45-74`. Second,
in-process `runCli()` calls cannot overlap; await each one or pass
`{ subprocess: true }`.

## Code

Every module opens with a docblock that argues the design decision rather than
describing the API, usually tagged with the release wave and unit id, and it names
the rejected alternative and the measurement that refuted it. Several modules
declare their single reason to change. Inline comments explain why the code is not
the obvious thing.

`interface` for shapes (1642 exports against 355 `type`, which is reserved for
unions), every field `readonly`, `ReadonlyArray`/`ReadonlyMap`/`ReadonlySet`, and
2145 `Object.freeze` calls in `src/`.

Closed vocabularies are always four pieces together: a frozen object with
camelCase keys and snake_case values, a derived union type, a members array, and a
type guard. Canonical form at `negative-recall.ts:115-143`.

No shared error module: roughly forty per-module exported `Error` subclasses, each
with a stable `code`, structured readonly fields, `this.name` set, and a message
that names the fix. Config errors go through `BrainConfigError(message,
"block.sub_key", source)`.

Named exports only (exactly one `export default` in all of `src/`).
`verbatimModuleSyntax` means `import type`; `allowImportingTsExtensions` means
relative imports carry `.ts`. `strict`, `noUncheckedIndexedAccess`,
`noImplicitOverride`, `noFallthroughCasesInSwitch`, `isolatedModules`. Format
width 100.

## CI gates, in order

`validate` job: checkout, Bun, Python 3.11, `bun install --frozen-lockfile`,
`bun run sync-version:check`, the OpenClaw bundle byte-diff, `link-ratchet:check`,
`check:paths` (report only), `fmt:check`, `lint`, `typecheck`, `bun test`,
`python -m unittest discover -s tests/python -v`, `python -m compileall -q
plugins/hermes`.

Version sync runs before everything else, so a forgotten bump fails the pull
request before lint or tests start.

`bun run validate` is typecheck, lint and test; it does **not** include
`fmt:check`. The pre-commit hook does, so format separately.

Practical pre-push sequence: `bun run fmt && bun run lint && bun run typecheck &&
bun run build:openclaw && bun run sync-version && bun run link-ratchet:check &&
bun test`.

## Architectural guard tests

`tests/core/architecture/write-site-census.test.ts`. A module is in scope if it
lives under `src/core/brain/`, `src/core/search/`, `src/cli/brain/`, `src/mcp/`,
or is `vault.ts`/`fs-atomic.ts`, or imports any relative `paths.ts` from anywhere.
Fifteen sync fs calls plus their promise twins plus `Bun.write` count as writes;
`mkdirSync` is deliberately excluded. The preferred fix is routing through a
shared writer; otherwise add an entry with categories from a closed vocabulary, a
`calls` array listing exactly the calls made, sorted, and a non-empty reason. Four
assertions fail you, including an exclusion whose file no longer writes directly,
and a `calls` array that drifts in either direction.

`tests/core/architecture/import-cycles.test.ts`. Zero cycles in `src/`.
`import type` is an edge; `await import()` is not, and deferred import is the
sanctioned cure. A new leaf module must import nothing from the layer above it.

`tests/core/architecture/verdict-vocabulary-census.test.ts`. Every closed
vocabulary in the "silence is not an answer" family, or whose values are copied
out of TypeScript into a tool schema or a persisted file, must register as
`{name, values, members, guard}` with a comment. The audit demands a frozen values
object, no duplicates, members and values in bijection, a guard that accepts every
member and rejects the empty string.

Others that fail a pull request: `tests/core/layering.test.ts:45` bans

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itechmeat/open-second-brain](https://github.com/itechmeat/open-second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
