---
trigger: always_on
description: Guidance for coding agents (and humans) working in this repository. For setup,
---

# AGENTS.md

Guidance for coding agents (and humans) working in this repository. For setup,
PR workflow, and release process, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## About Eve

Eve is a filesystem-first framework for durable backend AI agents. You author
an agent as a directory on disk — instructions, skills, tools, connections,
channels, subagents, and schedules are all files — and Eve compiles and runs it.
See the [README](./README.md) for the full overview and
[`docs/`](./docs) for user-facing documentation.

## Repository layout

- `packages/eve` — the framework and `eve` CLI (the main package)
- `packages/eve-catalog` — internal, unpublished library
- `apps/fixtures` — shared agent fixtures used by e2e, TUI smoke tests, and local dev
- `apps/frameworks`, `apps/templates`, `apps/docs` — framework integrations, templates, docs site
- `docs` — published documentation content
- `e2e/` — fixture-owned `eve eval` end-to-end tests

## Commands

```sh
pnpm install            # install workspace dependencies
pnpm build              # build all packages
pnpm dev                # watch-mode build + weather fixture on an available local port

pnpm typecheck          # TypeScript across the workspace
pnpm lint               # oxlint (auto-fixes)
pnpm fmt                # oxfmt
pnpm guard:invariants   # mechanical code-invariant checks (runs in CI)
pnpm docs:check         # docs frontmatter and nav validation

pnpm test               # unit + integration
pnpm test:unit          # unit tests (<3s)
pnpm test:integration   # integration tests (<10s)
pnpm test:scenario      # scenario tests (2–5 min; requires pnpm build first)
pnpm test:e2e           # fixture-owned eve eval suites
pnpm test:tui           # TUI smoke scripts (not e2e)
```

Verify while iterating, not only at the end: run `pnpm test:unit` after each
meaningful edit, `pnpm test:integration` before declaring a unit of work done,
and `pnpm test:scenario` when touching the compiler, runtime, dev server, CLI,
or scenario fixtures. Nothing is done until `typecheck`, `lint`, `fmt`,
`build`, and the tests all pass. Documentation-only changes are exempt.

## Coding principles

1. **Public APIs need docs and tests.** Exported functions, classes, and
   important public types get documentation comments and at least one test.

2. **Small modules over big helpers.** Favor composable primitives with narrow
   responsibilities. Split files that carry multiple concerns.

3. **Wrap third-party dependencies.** Do not expose third-party APIs as Eve
   public APIs. Wrap them in Eve-owned surfaces so internals can change freely.
   Add runtime `dependencies` only as a last resort: prefer vendoring code or
   generated artifacts into the repository and listing the source package under
   `devDependencies`. The `eve` package should aim to keep `nitro` as its only
   runtime dependency. This keeps Eve installs as small as possible and avoids
   exposure to hijacked nested dependencies that are not pinned directly in the
   main lockfile.

4. **Pre-1.0: prefer breaking changes.** Favor correctness and simplicity over
   backwards compatibility. No legacy fallback logic.

5. **Derive names from file paths.** Connection names, tool names, and similar
   identifiers come from the filesystem path (e.g.
   `agent/connections/linear.ts` → `"linear"`). Do not add redundant `name`
   fields to definitions.

6. **Name definitions for the protocol they target.** Use
   `defineMcpClientConnection`, not `defineConnection`.

7. **All runtime functionality lives in the `eve` package.** Never rely on
   emitted or generated code for runtime behavior.

8. **Comment why, not what.** Default to no comment; well-named code is the
   documentation. Comment only what the code cannot say itself — a non-obvious
   why, an invariant, a surprising edge case. Public API docs (principle 1) are
   the exception.

Machine-checkable invariants are enforced by `pnpm guard:invariants`, which
runs in the CI lint job. If the guard fails, fix the violation rather than
editing the baseline — baselines may only shrink.

## Testing

Tests belong in one of four tiers. Pick the tightest tier that can express the
assertion:

- **Unit** (`src/**/*.test.ts`): pure logic, colocated. No filesystem writes,
  subprocesses, or real network calls.
- **Integration** (`src/**/*.integration.test.ts`): multiple modules in memory.
- **Scenario** (`src/**/*.scenario.test.ts`, `test/scenarios/`): real
  subprocess, HTTP port, or bundler.
- **E2E** (`e2e/fixtures/*/evals/`, plus `apps/fixtures/weather-fixture/evals/`):
  fixture-owned `eve eval` suites. Local e2e is just `eve eval`; Vercel e2e is
  fixture-local prebuilt deploy followed by `eve eval --url`.

**Running a single file or filtered test: always pass the tier config.** Only
the `vitest.<tier>.config.ts` files alias `#*` imports to `./src`; a bare
`vitest run <path>` resolves them to compiled `./dist` output, so you end up
testing stale builds. Use:

```sh
pnpm --filter eve exec vitest run --config vitest.unit.config.ts <path-or-pattern>
# or vitest.integration.config.ts / vitest.scenario.config.ts for those tiers
```

Add `-t "<name>"` to filter by test name. If you touched anything under

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel/eve](https://github.com/vercel/eve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
