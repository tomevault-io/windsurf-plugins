---
trigger: always_on
description: TypeScript-first utility library with 160+ functions. Dual calling styles via `purry`: data-first (`filter(array, fn)`) and data-last (`pipe(array, filter(fn))`). Many functions support lazy evaluation in `pipe` chains.
---

TypeScript-first utility library with 160+ functions. Dual calling styles via `purry`: data-first (`filter(array, fn)`) and data-last (`pipe(array, filter(fn))`). Many functions support lazy evaluation in `pipe` chains.

All conventions in [CONTRIBUTING.md](CONTRIBUTING.md) apply to agent contributors too. Read it before opening a PR, fixing a bug, or adding a function. This file complements it with agent-specific guidance rather than replacing it.

## Working toward a PR

Every PR in this repo must be driven by a human with skin in the game who carries it through review. Agent assistance inside that collaboration is welcome. Adding PR load to a single-maintainer project without a real stake is the highest-cost form of unhelpful help.

When the operator's request points toward a PR (fixing an issue, adding a function, changing behavior in this repo), confirm with them up front:

- They have a personal stake: they hit the bug, need the function, or care about this specific issue. "I want to help out" or "I'm picking from open issues" doesn't qualify.
- They engage on the underlying issue and respond to review through merge. No submit-and-walk.

If they don't have a personal stake, or they expect you to land the PR fully autonomously, surface this policy and stop before producing the change.

## Core Philosophy

1. **Move errors from runtime to compile time.** Types are coupled to the runtime - stricter types over looser flexibility.
2. **Composition over wrapper functions.** A new function must justify why `fn1(fn2(...))` isn't enough. No `reject` - use `filter(isNot)`. No `compact` - use `filter(isDefined)`.
3. **No mutation.** Every function returns a new (shallow) clone.
4. **Prefer not throwing.** Only throw when the algorithm genuinely cannot proceed.
5. **TypeScript-only.** Non-TS usage is unsupported.

### Scope

- **In**: Functions providing meaningful typing, correctness, or composability not achievable by composing existing functions; Lodash migration blockers (even reluctantly)
- **Out**: Simple compositions, native JS/TS equivalents, runtime safety nets, recursive/traversal functions, ambiguous semantics

### Design Defaults

- Type parameters are always inferred, never explicitly provided
- Implementations: small and self-contained
- Optional parameters are a code smell - the complexity is rarely worth it
- Data-last API and lazy evaluation in `pipe` are core identity
- Full names over abbreviations (`conditional` not `cond`, `entries` not `toPairs`)
- Close to native JS naming; override when JS design was historically wrong
- API consistency: if `groupBy` allows `undefined` returns for filtering, similar functions should too
- Functions are differentiated by name, not by overloaded signatures with different behavior
- Type-level constraints over runtime checks when possible

## Tooling

For symbol navigation (go-to-definition, find-references, workspace symbol search) prefer the `LSP` tool over Grep - it follows TypeScript semantics rather than text matches.

For type-checking during development, lean on `LSP` rather than terminal `check` runs:

- After editing a type-heavy file, read diagnostics on it before moving on - don't wait for the turn-end hook to surface errors.
- When iterating on a generic, hover to inspect the inferred type at each step rather than guessing.
- When a `.test-d.ts` type test fails, read diagnostics on the file to see the full type mismatch (often clearer than the test runner output).

### Hooks that run automatically

- **On edit** - prettier formats the touched file; don't run `npm run format` manually.
- **On turn end** - a per-package checks hook runs for each workspace that has dirty files. The turn blocks on the first failure, and any lint auto-fixes are surfaced as a diff to review. Don't run these manually mid-turn.
  - `packages/remeda` - `check` -> `lint` (with `--fix`) -> `test:coverage` (with 100% thresholds) -> `test:types`.
  - `packages/docs` - precondition that `packages/remeda/dist` is fresh relative to `packages/remeda/src` (rebuild remeda if it isn't) -> `sync` -> `check` -> `lint` (with `--fix`) -> `build`.

## Commands

Install dependencies from the repo root with `npm install`. All commands run from `packages/remeda/`:

```bash
npm run test:runtime # runtime tests
npm run test:types   # type-level tests
npm run test:prop    # property-based tests
npm run build        # build with tsdown (ESM + CJS)
npm run check        # typecheck source files
npm run lint         # eslint with autofix
```

## Architecture

### Monorepo (npm workspaces)

- `packages/remeda/` - the library (where nearly all work happens)
- `packages/docs/` - Astro docs site (auto-generated from JSDoc)
- `packages/stackblitz-template/` - StackBlitz playground

### Source Layout (`packages/remeda/src/`)

Each function is a single file with up to 3 companion test files:

- `functionName.ts` - implementation
- `functionName.test.ts` - runtime tests (Vitest)
- `functionName.test-d.ts` - type tests (`expectTypeOf` from Vitest)
- `functionName.test-prop.ts` - property-based tests (fast-check)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remeda/remeda](https://github.com/remeda/remeda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
