---
trigger: always_on
description: This file guides AI assistants working on this repository. Prefer durable,
---


This file guides AI assistants working on this repository. Prefer durable,
spec-aligned, maintainable changes over local fixes or convenience hacks.

## Required workflow

### Spec-governed work

Use the spec pipeline for major features, architectural changes,
schema/API boundaries, cross-package coordination, dependency changes, or
other roadmap-level capabilities.

Before coding spec-governed work, you must:

1. Read `docs/specs/INDEX.md`.
2. Validate `docs/specs/dependencies.yaml`.
3. Confirm the target spec status. Do not implement major feature work
   from `planning`; it must be `approved` first.
4. Keep `INDEX.md` and `dependencies.yaml` in sync when adding or
   changing spec work.
5. Re-check integration contracts and invariants in `INDEX.md` before
   finishing.

Skip the planning tier for routine bug fixes, narrow refactors, tests,
and docs-only edits that stay within existing contracts.

### Incident -> guardrail loop

A bug fix is not complete until it adds at least one durable prevention
mechanism:

1. Regression test, invariant check, or CI guard.
2. Spec/index/dependency update if behavior or sequencing changed.
3. `AGENTS.md` rule or checklist refinement if the failure was process
   related.

Every incident should make the same failure mode harder to repeat.

## Common PR failure modes

Prevent these proactively:

1. **Scoping leaks**
   - Thread `agent_id` through every read/write touching user data.
   - Thread `visibility` where relevant.
   - Never hardcode `"default"` for scoped paths.

2. **Validation and bounds gaps**
   - Validate external inputs and config at boundaries.
   - Clamp counters, latency values, and limits to sane non-negative
     ranges.
   - Reject out-of-range values with clear errors.

3. **Silent failures / weak fallbacks**
   - Do not swallow errors.
   - Log with context and return structured failures.
   - For retries or refresh loops, enforce timeout floors,
     single-flight/serialization, and timer cleanup.

4. **Doc/spec drift**
   - Update behavior, API, schema, and status docs in the same PR.
   - Keep `docs/API.md`, `docs/specs/INDEX.md`, and
     `docs/specs/dependencies.yaml` accurate when affected.
   - Root docs duplicated into `docs/` are generated artifacts. Edit the
     root source, then run `bun scripts/sync-root-docs.ts`. Do not hand-edit
     `docs/CONTRIBUTING.md` or `docs/ROADMAP.md`.

5. **Duplication / parity drift**
   - Do not duplicate constants, maps, dependency types, descriptions,
     or config defaults across files.
   - Extract shared sources of truth.

6. **Missing regression tests**
   - Every bug fix needs a test that would fail before the fix.
   - Add edge-case tests for scoping, invalid inputs, timer lifecycle,
     and fallback behavior.

7. **Security/auth oversights**
   - Admin, refresh, and mutation endpoints need explicit permission
     checks.
   - Add rate limiting to expensive or abuse-prone paths.

8. **Code hygiene misses**
   - Run lint and remove dead vars/imports before review.
   - Keep comments aligned with implementation.
   - In `packages/cli/dashboard/`, never run broad Biome autofix
     blindly. Scope it narrowly, inspect the diff, and rerun:

```bash
cd packages/cli/dashboard && bun run check
```

9. **Publish/install integrity drift**
   - Publishable packages must not ship runtime dependencies on
     unpublished workspace packages.
   - Validate publish manifests after version rewriting and before npm
     publish.

## PR checklist

Before opening a PR, verify:

- Spec alignment is checked (`INDEX.md` + `dependencies.yaml`) when the
  change touches spec-governed behavior.
- Agent scoping is correct on all changed data queries.
- Input/config validation and bounds checks were added.
- Error handling and fallback paths are tested.
- Security checks exist on admin or mutation endpoints.
- Docs were updated for API, spec, schema, or status changes.
- Publish manifests were validated if the PR changes a package that gets
  published to npm.
- Each bug fix has a regression test.
- Lint, typecheck, and tests pass locally.

## Priorities

1. Performance.
2. Reliability.
3. Predictable behavior under load and failure.
4. Correctness and robustness over short-term convenience.
5. Long-term maintainability and shared abstractions over local hacks.
6. Duplicate logic is a smell. Extract shared code where it clarifies.
7. Do not be afraid to change existing code when that produces a better
   system.

## Workspace commands

```bash
bun install
bun run build
bun run dev
bun test
bun run lint
bun run format
bun run typecheck
bun run build:publish
bun run version:sync
bun run dev:web
bun run deploy:web
```

`bun run build` must respect this order:

```text
build:core -> build:connector-base -> build:opencode-plugin -> build:native
-> build:oh-my-pi-extension -> build:connector-oh-my-pi
-> build:pi-extension -> build:connector-pi -> build:deps
-> build:signetai
```

`@signet/pi-extension-base` is a source-only shared package with no
standalone build step. `build:oh-my-pi-extension` and `build:pi-extension`
consume it directly from workspace source.

Run a single test file directly with:

```bash
bun test packages/daemon/src/pipeline/worker.test.ts
```

## Package map


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Signet-AI/signetai](https://github.com/Signet-AI/signetai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
