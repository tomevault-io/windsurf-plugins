---
trigger: always_on
description: | Tool                                   | Purpose                                                              |
---

# Conventions

## Tooling

| Tool                                   | Purpose                                                              |
|----------------------------------------|----------------------------------------------------------------------|
| TypeScript 5 (`@tada5hi/tsconfig`)     | Type checking via `npm run build:types` (`tsc`, `noEmit: true`); also used by editors/IDEs. |
| tsdown (Rolldown + Oxc)                | `npm run build:js` — bundles `src/index.ts` to `dist/index.{cjs,mjs}` and emits `dist/index.d.{cts,mts}` declarations. |
| Vitest 4                               | Test runner (config at `test/vitest.config.ts`).                     |
| ESLint 10 (`@tada5hi/eslint-config`) + `typescript-eslint` | Linting via flat config (`eslint.config.js`).    |
| Husky 9                                | Git hooks (currently only `commit-msg`).                             |
| commitlint + `@tada5hi/commitlint-config` | Validates Conventional Commits on every commit.                   |
| release-please                         | Automates version bumps, CHANGELOG, and GitHub releases.             |
| `tada5hi/monoship@v2`                  | GitHub Action used by the release workflow to publish to npm.        |
| Codecov                                | Receives coverage upload from the release workflow.                  |

## Workflow

After making source changes:

1. `npm run build` if you touched anything in `src/` (especially before manually testing the dist output).
2. `npm run lint` (or `npm run lint:fix`).
3. `npm run test` — Vitest runs against `src/` directly, no build needed for tests.
4. Commit with a Conventional Commits message (commitlint will reject otherwise).

The build step is not bound to commit time; the release workflow runs `npm run build` before publishing (which only happens from CI).

## Code Style

- **Module format**: ESM source (`"type": "module"`, `import`/`export`). The build produces dual CJS + ESM.
- **Indentation**: 4 spaces (enforced by `.editorconfig`). LF line endings, final newline, no trailing whitespace.
- **Linting**: ESLint 10 flat config in `eslint.config.js`, extending `@tada5hi/eslint-config` with project-specific overrides:
  - `class-methods-use-this`, `no-continue`, `no-shadow`, `no-use-before-define`, `no-useless-escape`, `no-nested-ternary` — **off**.
  - `@typescript-eslint/no-unused-vars`, `@typescript-eslint/no-use-before-define` — **off**.
  - `import/no-extraneous-dependencies` allows devDependencies in `test/**`, `*.spec.{js,ts}`, `tsdown.config.ts`, `eslint.config.js`, and `commitlint.config.mjs`.
  - `dist/**` and `**/*.d.ts` are ignored.

## Naming Conventions

- **Type exports** live in `src/type.ts` and use `Options`, `Merger`, `Merger*` (`MergerSource`, `MergerResult`, `MergerContext`, `MergerSourceUnwrap`) — no `I`-prefix.
- **Predicates** are prefixed `is*` and return `item is T` where useful (`isObject`, `isSafeKey`, `isEqual`).
- **Builders** are prefixed `build*` (`buildOptions`); **togglers** are prefixed `toggle*` (`togglePriority`).
- **Factories** are prefixed `create*` (`createMerger`).
- **Files**: kebab-case is unused — all source files are single-word lowercase (`module.ts`, `presets.ts`, `array.ts`, etc.) sitting next to their barrel `index.ts`.

## File Organization

- `src/type.ts` is the single home for **all exported types**. Do not split types across multiple files unless they are unambiguously internal to one module.
- `src/utils/` is a flat directory of single-concern utility files plus an `index.ts` barrel that re-exports everything.
- Public API is **only** what `src/index.ts` re-exports — adding a new file under `src/` does not automatically expose it; it must be re-exported from a reachable barrel.

## Pre-commit Hooks

Husky runs **`commit-msg`** only (`.husky/commit-msg`):

1. `commitlint --edit` validates the commit message against `@tada5hi/commitlint-config`.

There is no `pre-commit` lint or test hook. Lint/test failures are caught in CI, not locally on commit.

## Commit Convention

Commits follow **Conventional Commits** (`@tada5hi/commitlint-config`). Format:

```
<type>(<optional scope>): <subject>

[optional body]

[optional footer]
```

Common types: `feat`, `fix`, `chore`, `build`, `ci`, `docs`, `refactor`, `test`. The Dependabot config also produces commits with `fix(<scope>)` and `build(<scope>)` prefixes — keep those conventions if you author similar changes manually.

**Do not** add `Co-Authored-By: Claude ...` (or any AI-attribution) trailers to commits in this repo.

## TypeScript

`tsconfig.json` extends `@tada5hi/tsconfig` with:

- `target: ES2022`, `module: ESNext`, `moduleResolution: bundler`
- `lib: ["ESNext", "DOM"]` (DOM is needed for `globalThis` typing in `src/utils/clone.ts`)
- `noEmit: true`, `allowImportingTsExtensions: true` — no emission from `tsc`; tsdown handles all JS/`.d.ts` output.
- `noUncheckedIndexedAccess: false` — re-disabled (the base config enables it) because the merge hot path uses indexed `for` loops over `Object.keys()`, which would otherwise return `string | undefined` from each index and force pervasive non-null assertions.

## Build Output

`npm run build` runs `tsdown` and produces, in `dist/`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tada5hi/smob](https://github.com/tada5hi/smob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
