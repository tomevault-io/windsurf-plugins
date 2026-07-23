---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

- **Package manager**: pnpm 9.15.9 with strict isolated `node_modules` (default). Pinned via `packageManager` in root `package.json` and activated through Corepack. Scripts assume `pnpm`.
- **Node**: `.nvmrc` is `24.16.0`; `engines.node` in root `package.json` is `>=24.16.0`; CI runs the unit-test matrix on Node `22.x` and `24.x` and every other job off `.nvmrc`.
- **Monorepo**: Turborepo + pnpm workspaces. Workspaces declared in `pnpm-workspace.yaml`: `packages/*`, `targets/*`, `demo`, `docs`.
- **Bundler**: `tsdown` per package, sharing `tsdown.config.base.mjs`. The library is **ESM-only** — each package emits a single modern ESM bundle (`dist/<name>.modern.mjs`) plus its `.d.mts` types. No CJS, no legacy/pre-compiled variants.
- **Tests**: Vitest in browser mode (Chromium via Playwright) for both unit and E2E projects, `vitest-browser-react` for rendering, `tsc --noEmit` (types). Root config: `vitest.config.ts`.
- **Lint/format**: [oxlint](https://oxc.rs/docs/guide/usage/linter) (root `.oxlintrc.json`) + [oxfmt](https://oxc.rs/docs/guide/usage/formatter) (root `.oxfmtrc.json`). Husky `pre-commit` runs `oxfmt --check`; `commit-msg` runs commitlint with `@commitlint/config-conventional`. VS Code users should install the [oxc extension](https://marketplace.visualstudio.com/items?itemName=oxc.oxc-vscode) — see `.vscode/extensions.json`.

## Common commands

| Task                                 | Command                                                                                       |
| ------------------------------------ | --------------------------------------------------------------------------------------------- |
| Install                              | `pnpm install --frozen-lockfile`                                                              |
| First-time browser install           | `pnpm exec playwright install chromium`                                                       |
| Build all packages                   | `pnpm build` (turbo, respects `^build` deps)                                                  |
| Build everything except docs         | `pnpm build-ci`                                                                               |
| Watch-build all packages in parallel | `pnpm dev`                                                                                    |
| Run docs / demo dev servers          | `pnpm docs:dev` / `pnpm demo:dev`                                                             |
| Full test suite                      | `pnpm test` (ts + unit + e2e)                                                                 |
| Unit tests                           | `pnpm test:unit`                                                                              |
| Watch unit tests                     | `pnpm vitest --project unit`                                                                  |
| Single test file                     | `pnpm vitest run packages/core/src/SpringValue.test.ts`                                       |
| Filter by test name                  | `pnpm vitest run -t "interpolation"`                                                          |
| Headed (visible browser) debugging   | `pnpm vitest --project unit --browser.headless=false <file>`                                  |
| Coverage                             | `pnpm test:cov` (thresholds: 80% statements / 74% branches / 71% functions / 82% lines)       |
| Type-check                           | `pnpm test:ts`                                                                                |
| Parallax E2E                         | `pnpm test:e2e` (programmatically serves `packages/parallax/test`, runs Vitest browser specs) |
| Lint                                 | `pnpm lint` (turbo across packages)                                                           |
| Format                               | `pnpm format` / `pnpm format:check`                                                           |

Note: `vitest.config.ts` aliases `@react-spring/*` to the package source under `packages/*/src/index.ts`, so unit tests run **without** a prior build. Anything outside Vitest (docs, publish-ci) needs `pnpm build` first.

Strict isolation: `node_modules` is non-hoisted, so a workspace can only `import` packages it declares in its own `package.json`. If you see a `Cannot find module 'foo'` error after adding an import, add `foo` to that workspace's `dependencies` / `peerDependencies` / `devDependencies` — do not add a hoist rule.

## Architecture

The library is a layered monorepo. Read packages bottom-up — each layer is target-agnostic until you reach `targets/*`. There is no umbrella package: consumers install a target directly (`@react-spring/web` or `@react-spring/three`).

```
          targets/web     targets/three
              │                 │
              └────────┬────────┘
                       │
              packages/core    ─── declarative API: hooks, components, SpringValue, Controller, SpringRef
                       │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pmndrs/react-spring](https://github.com/pmndrs/react-spring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
