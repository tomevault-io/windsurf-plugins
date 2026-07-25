---
trigger: always_on
description: This repository is **Accord Project markdown-transform** — a TypeScript npm-workspaces monorepo for parsing and transforming Markdown, CommonMark, CiceroMark, and TemplateMark.
---

# GitHub Copilot Instructions for Markdown-Transform

This repository is **Accord Project markdown-transform** — a TypeScript npm-workspaces monorepo for parsing and transforming Markdown, CommonMark, CiceroMark, and TemplateMark.

## Project context

- Runtime: Node.js `>=22`
- Package manager: `npm` (workspace root + package-level scripts)
- Language: **TypeScript** (target `ES2020`, `module: commonjs`). Source lives in `packages/*/src/`; compiled `.js` + `.d.ts` are emitted to `packages/*/lib/`.
- Build: `tsc` per package (config extends `tsconfig.base.json`).
- Linting: ESLint with `@typescript-eslint` (4-space indent, single quotes, semicolons).
- Unit testing: **Jest 29 + ts-jest** across every package. The legacy mocha+chai suites were removed during the TS migration.
- Browser E2E: **Playwright** under `e2e/` exercises the UMD bundles in headless Chromium.
- Bundling: `webpack 5` produces UMD bundles for `markdown-html`, `markdown-template`, `markdown-transform` (the three user-facing entry points). The other packages are CommonJS library deps consumed via bundlers.
- CI: GitHub Actions matrix on Ubuntu, macOS, and Windows for unit tests; Ubuntu-only for Playwright e2e.

## Repository layout

- `packages/` — eight publishable packages:
  - `markdown-common`
  - `markdown-cicero`
  - `markdown-template`     *(also UMD)*
  - `markdown-html`         *(also UMD)*
  - `markdown-it-cicero`
  - `markdown-it-template`
  - `markdown-cli`
  - `markdown-transform`    *(umbrella, also UMD)*
- `e2e/` — browser end-to-end tests (Playwright). Not published.
- `scripts/` — repo-level utilities (model generation, version bumping, coverage aggregation).
- `tsconfig.base.json` — shared compiler options inherited by every package.

Concerto models for CommonMark/CiceroMark/TemplateMark are downloaded by `scripts/external/getExternalModels.js` (run via `npm run models:get` / triggered as `postinstall`) and emitted as TS into `packages/markdown-common/src/externalModels/`. Treat those files as generated.

## Non-negotiable contribution requirements

1. **DCO sign-off is required on every commit**
   - Use `git commit --signoff` (or `-s`).

2. **Commit message format follows Accord Project conventions**
   - Prefer: `type(scope): description`
   - Examples seen in this repo:
     - `fix: update broken CI badge to use GitHub Actions workflow URL`
     - `chore(deps): update package dependencies`
     - `chore(actions): publish v1.0.0 to npm`

3. **Do not skip tests for behavior changes**
   - Add or update unit tests when changing logic; add a Playwright e2e test if the change affects the browser bundle surface.

## Coding standards (repo-specific)

- TypeScript only for new code. Avoid reintroducing `.js` files in `src/`.
- 4-space indentation, single quotes, semicolons.
- Prefer `const`/`let` (no `var`), keep braces (`curly`), strict equality (`eqeqeq`).
- The TS config is pragmatic — `strict: false`, `noImplicitAny: false` — so visitor/AST code uses `any` liberally. That is intentional: don't tighten types in unrelated files while fixing something else.
- Don't add JSDoc that simply restates the function signature; reserve comments for non-obvious *why*.
- Prefer minimal, surgical diffs; avoid unrelated formatting churn.

## Build and test workflow

When changing code, run checks in this order:

1. `npm run build` — runs `tsc` per workspace (also rebuilds before tests via each package's `pretest`).
2. `npm test` — runs the full Jest suite across every package.
3. `npm run -w markdown-transform-e2e test` — Playwright browser tests; only needed if you changed source that ends up in a UMD bundle.
4. `npm run coverage` — coverage aggregation (only if investigating coverage).

For package-level iteration, `cd packages/<name>` and run `npm run build`, `npm test`, etc. directly. For the umbrella package, also run `npm run webpack` after `npm run build` to refresh the UMD bundle.

When migrating Concerto: `@accordproject/concerto-core` is on **v4**. `new ModelManager({ strict: true })` is no longer valid — drop the option, don't cast to `any`. The model manager defaults are equivalent in v4.

## Dependency management rules (critical)

These are based on merged PR review feedback in this repository:

1. **Avoid broad `npm audit fix`-style dependency churn**
   - Reviewers flagged PRs where this introduced unnecessary new dependencies and unexpected downgrades.
   - Keep dependency updates intentional and minimal.

2. **Do not introduce version downgrades unless explicitly justified**
   - If downgrading is required, explain why in PR description and comments.

3. **Avoid adding new dependencies without clear rationale**
   - Reviewers repeatedly asked "Why the new deps?" across multiple package manifests.
   - Prefer updating existing dependencies over adding new ones.

4. **For core Accord dependencies, prefer exact versions when the repo already pins exact versions**
   - PR feedback explicitly pushed back on moving `@accordproject/concerto-core` from exact to caret ranges.
   - Keep versions consistent with surrounding package conventions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [accordproject/markdown-transform](https://github.com/accordproject/markdown-transform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
