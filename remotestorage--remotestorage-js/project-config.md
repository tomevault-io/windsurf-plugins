---
trigger: always_on
description: - This file applies to the entire repository: `remotestorage/remotestorage.js`.
---

# Scope

- This file applies to the entire repository: `remotestorage/remotestorage.js`.
- Follow these instructions for all code changes, scripts, and tests.

# Build, Lint, Test

- Install dependencies: `npm ci` (CI) or `npm install` (local).
- TypeScript compile: `tsc` or `tsc -w` for auto-compile on changes.
- Development bundle/watch: `npm run dev`.
- Production bundle: `npm run build:release` (webpack production).
- Lint sources: `npm run lint` (verbose) or `npm run lint:quiet`.
- Lint Mocha specs: `npm run lint:specs` or `npm run lint:specs:quiet`.
- Full test suite: `npm test` (runs `tsc` and `scripts/test-all.sh`).
- Mocha unit tests: `npm run test:mocha`.
- Mocha watch: `npm run test:watch`.
- Typedoc docs (CI release step): `typedoc` via `npm run version`.

# Running a Single Test

- Mocha (preferred for new tests):
  - Single file: `npm run test:mocha -- test/unit/<name>.test.mjs`.
  - Single test by grep: `npm run test:mocha -- --grep "pattern"`.
  - Watch a file: `npm run test:watch -- test/unit/<name>.test.mjs`.
- Jaribu (legacy suites while being ported):
  - Single suite: `./node_modules/.bin/jaribu test/unit/<suite>-suite.js`.
  - Notes: Jaribu suites are older `.js` files like `test/unit/inmemorycaching-suite.js`.

# Continuous Integration

- GitHub Actions workflow: `.github/workflows/test-and-lint.yml`.
- Matrix Node versions: `18`, `20`.
- Steps: `npm ci`, `npm test` (Jaribu), `npm run test:mocha -- --exit`, lint tasks, `npm run build:release`.

# Project Overview

- Library entrypoints: TypeScript sources in `src/` compiled/bundled to `release/`.
- Docs: VitePress in `docs/` with Typedoc-generated API pages.
- Tests: Legacy Jaribu suites in `test/unit/*-suite.js`; Mocha/Chai specs in `test/unit/*.test.mjs`.

# Languages and Tooling

- TypeScript (target `es2015`, module `commonjs`), Mocha/Chai, Sinon, ESLint (`@typescript-eslint`), Webpack, Typedoc.
- Formatting: esformatter is only used for `src/sync.js` via `npm run format` (legacy). Prefer ESLint autofix for TS files.

# Code Style Guidelines

## Imports

- Use TypeScript ES module syntax: `import { Thing } from "./path";`.
- Prefer named imports; default imports only when the module exports default.
- Relative paths: keep them short and stable; avoid deep chained `../../..` where possible by reorganizing modules if needed.
- Do not use `require` in TypeScript files (`.ts`). The ESLint config warns for `@typescript-eslint/no-var-requires`.

## Formatting

- Indentation: 2 spaces. ESLint enforces `indent: ["error", 2]`.
- Curly braces required: `curly: 2`.
- Semicolons required: `semi: 2`.
- Arrow function spacing enforced: `arrow-spacing: 2`.
- Block spacing enforced: `block-spacing: 2`.
- No multi-line string literals using `\` concatenations: `no-multi-str: 2`.
- Console: only `console.warn` and `console.error` allowed. `no-console` blocks other methods.
- Bitwise operators not allowed: `no-bitwise: 2`.
- Equality: always use strict `===`/`!==` (`eqeqeq: 2`).

## Types

- Avoid `any` wherever possible (`@typescript-eslint/no-explicit-any: 1`). Prefer precise interfaces and type aliases.
- Prefer explicit return types on exported functions.
- Avoid unused variables and parameters (`@typescript-eslint/no-unused-vars: 1`).
- Avoid using variables before definition (`@typescript-eslint/no-use-before-define: 1`).
- Allow empty interfaces only if necessary (`@typescript-eslint/no-empty-interface: 1`).
- Shadowing is warned (`@typescript-eslint/no-shadow: "warn"`); refactor to avoid.
- Globals: the ESLint config defines browser and node environments. Don’t introduce implicit globals.

## Naming Conventions

- Use `camelCase` for variables, parameters, and functions.
- Use `PascalCase` for classes, types, and enums.
- Constructors/new-cap: ESLint enforces capitalization for constructor-like identifiers; exceptions include `Authorize`, `Discover` in legacy code.
- File names: prefer `kebab-case` or `lowercase` for `.ts` files; keep names descriptive and aligned with exported symbols.
- Constants: UPPER_CASE only for true compile-time constants; otherwise use `camelCase`.

## Error Handling

- Do not use `debugger` (`no-debugger: 2`).
- Fail fast on invalid inputs; validate arguments and throw specific errors.
- Use domain-specific error classes where available (e.g., `UnauthorizedError` in `src/unauthorized-error.ts`, `SyncError` in `src/sync-error.ts`, `SchemaNotFoundError` in `src/schema-not-found-error.ts`).
- Avoid swallowing errors; when catching, either handle or rethrow with context.
- Logging: prefer `console.warn`/`console.error` and keep messages actionable.

## Asynchrony and Side Effects

- Prefer `async/await` over raw Promise chains for readability.
- Make network/storage side effects explicit in function names and docs.
- Avoid shared mutable state; encapsulate in classes/modules.

# Testing Guidelines

- New tests: write Mocha/Chai specs in `test/unit/*.test.mjs`.
- Use `sinon` for stubs/mocks/spies as needed.
- Keep tests deterministic; avoid relying on timers or external services.
- For single-test debugging: use `--grep` or isolate a `describe.only`/`it.only` in local runs (revert before committing).
- Lint specs with `npm run lint:specs`.

# Documentation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remotestorage/remotestorage.js](https://github.com/remotestorage/remotestorage.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
