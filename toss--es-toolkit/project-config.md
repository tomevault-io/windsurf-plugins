---
trigger: always_on
description: This file gives LLM coding agents the context they need to contribute to es-toolkit. Read it before making changes, and follow it together with [`.github/CONTRIBUTING.md`](./.github/CONTRIBUTING.md) (and its translations).
---

# es-toolkit LLM coding agent instructions

This file gives LLM coding agents the context they need to contribute to es-toolkit. Read it before making changes, and follow it together with [`.github/CONTRIBUTING.md`](./.github/CONTRIBUTING.md) (and its translations).

## Project overview

es-toolkit is a modern, high-performance JavaScript utility library with a small bundle size and strong type annotations. It provides a curated set of utilities that are difficult to write with built-in JavaScript methods but are commonly needed in real-world projects, such as [`delay`](https://es-toolkit.dev/reference/promise/delay.html), [`windowed`](https://es-toolkit.dev/reference/array/windowed.html), [`keyBy`](https://es-toolkit.dev/reference/array/keyBy.html), [`mapValues`](https://es-toolkit.dev/reference/object/mapValues.html), [`camelCase`](https://es-toolkit.dev/reference/string/camelCase.html), and [`toSnakeCaseKeys`](https://es-toolkit.dev/reference/object/toSnakeCaseKeys.html).

The library is split into two entry points:

- **`es-toolkit`** — the strict, opinionated main library. Focused on the 85% use case with a simple interface, no complex options, and modern JavaScript semantics.
- **`es-toolkit/compat`** — a Lodash-compatible variant intended to make migration from Lodash easy. Compat is feature-complete; only behavior fixes against Lodash are accepted, not new functions.

es-toolkit is a zero-dependency library and ships with no runtime dependencies.

## Development environment

- **Primary environment**: Node.js v24 (see `.nvmrc` for the exact version)
- **Recommended editor**: Visual Studio Code
- **Package manager**: Yarn v4, installed via [Corepack](https://yarnpkg.com/getting-started/install). See [the package manager section in CONTRIBUTING.md](./.github/CONTRIBUTING.md#package-manager) for details.

To get started:

```bash
corepack enable
yarn install
```

Common commands:

```bash
yarn vitest run                   # Run all tests
yarn vitest run src/array/chunk   # Run tests for a specific function
yarn lint                         # ESLint
tsc --noEmit                      # Typecheck
```

## Repository structure

```
src/
  {category}/{fn}.ts          Implementation for the main `es-toolkit` library
  {category}/{fn}.spec.ts     Unit tests (vitest)
  compat/{category}/{fn}.ts   Lodash-compatible variant under `es-toolkit/compat`
  index.ts                    Top-level barrel re-exports
  {category}/index.ts         Per-category barrel re-exports
docs/
  reference/{category}/{fn}.md          English reference docs
  ko/reference/{category}/{fn}.md       Korean reference docs
  ja/reference/{category}/{fn}.md       Japanese reference docs
  zh_hans/reference/{category}/{fn}.md  Simplified Chinese reference docs
benchmarks/                   Vitest benchmark suite
tests/types/                  Type tests comparing `es-toolkit/compat` with `@types/lodash` (run with `yarn workspace type-tests test`)
.github/CONTRIBUTING.md       Contribution guide (English / Korean / Simplified Chinese)
CHANGELOG.md                  User-facing changelog
```

Categories include `array`, `function`, `math`, `object`, `predicate`, `promise`, `set`, `string`, `util`, `error`, and `map`.

## Principles

es-toolkit values **performance**, **simplicity**, and **detailed documentation** over a wide surface area of features and options.

- **Performance**: every function should match or beat alternative libraries. New features must come with benchmarks.
- **Simplicity**: provide the simplest interface for the 85% use case; do not add complex options for every edge case.
- **Don't reimplement modern JS**: skip functions that are already covered by built-ins (`Array.isArray`, `Number.isNaN`, `Math.min`, `typeof value === 'number'`, etc.) or by TC39 proposals at Stage 3 or above.
- **Accurate types**: match the inference behavior of TypeScript's `strict` mode.
- **Two entry points, two policies**: `es-toolkit` is the opinionated API; `es-toolkit/compat` mirrors Lodash for migration. Compat is feature-complete — only behavior fixes against Lodash are accepted, not new functions.

For the full text, see [Section 1 of CONTRIBUTING.md](./.github/CONTRIBUTING.md#1-our-design-principles).

## Coding conventions

- Prefer `for` loops over `reduce`. Local mutability is fine.
- Prefer built-in JavaScript over custom helpers (`Array.isArray()` over `isArray()`, `typeof value === 'string'` over `isString()`).
- Use short type parameter names: `T` for elements, `K` for keys, `E` for errors.
- Use `readonly T[]` for array parameters that are not mutated.
- Write detailed JSDoc with `@template`, `@param`, `@returns`, `@throws`, and `@example` blocks.

For the full text, see [Section 2 of CONTRIBUTING.md](./.github/CONTRIBUTING.md#2-coding-conventions).

## Development workflow

When adding or changing a function, follow this order:

1. **Add the implementation** under `src/{category}/{fn}.ts` (or `src/compat/{category}/{fn}.ts` for compat-only fixes). Re-export it from `src/{category}/index.ts` and `src/index.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [toss/es-toolkit](https://github.com/toss/es-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
