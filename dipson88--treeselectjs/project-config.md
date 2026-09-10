---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

pnpm monorepo for `treeselectjs` — a vanilla TypeScript multi-select/tree-select component — plus thin React and Vue wrapper packages that re-export it. The wrappers have **no independent implementation**; they instantiate the core `Treeselect` class and sync props/refs to it. All actual behavior (rendering, keyboard nav, selection logic, search, positioning) lives in `packages/treeselectjs`.

```
packages/
  treeselectjs/          # core TS library — source of truth
  react-treeselectjs/    # React wrapper (depends on treeselectjs via workspace:*)
  vue-treeselectjs/      # Vue 3 wrapper (depends on treeselectjs via workspace:*)
```

## Commands

Run from repo root unless noted. Use `pnpm --filter <pkg> run <script>` to target one package (e.g. `pnpm --filter treeselectjs run jest:run`).

```bash
pnpm install               # install all workspace deps
pnpm build                 # build all packages (pnpm -r build)
pnpm dev                   # dev mode for all packages in parallel
pnpm check                 # biome check --write for all packages
pnpm changeset             # add a changeset before releasing
pnpm version-packages       # bump versions from changesets
pnpm release                # build + changeset publish
```

### Core package (`packages/treeselectjs`)

```bash
pnpm --filter treeselectjs run dev              # vite dev server, port 5173 (demo app)
pnpm --filter treeselectjs run typecheck        # tsc --noEmit across app/jest/cypress tsconfigs
pnpm --filter treeselectjs run jest:run         # unit tests (jsdom)
pnpm --filter treeselectjs run jest:watch
pnpm --filter treeselectjs run cypress:open     # interactive e2e (needs dev server running)
pnpm --filter treeselectjs run cypress:run      # headless e2e (needs dev server running)
pnpm --filter treeselectjs run test             # typecheck && jest:run && cypress:run
pnpm --filter treeselectjs run lint             # biome lint
pnpm --filter treeselectjs run lint:fix
```

Run a single Jest test file:
```bash
pnpm --filter treeselectjs exec jest __tests__/jest/tests/core-props/isSingleSelect.test.ts
```

Run a single Cypress spec (dev server must already be serving on 5173):
```bash
pnpm --filter treeselectjs exec cypress run --spec "__tests__/cypress/e2e/base/search.cy.ts"
```

React/Vue packages only have `dev`, `build`, `lint`, `check` (`build` = `tsc && vite build` for React, `vue-tsc && vite build` for Vue) — they have no test suites of their own; their behavior is exercised by the core package's Jest/Cypress suites.

## Architecture (core `treeselectjs`)

`src/treeselectjs.ts` exports the default `Treeselect` class, the single public entry point. It owns:
- All public props/methods from `ITreeselectParams`/`ITreeselect` (`src/treeselectTypes.ts`).
- Two internal sub-components it composes and wires together via callbacks (not events): `TreeselectInput` (`src/input/index.ts`) and `TreeselectList` (`src/list/index.ts`). `Treeselect` never touches their internals directly — it passes constructor callbacks (`inputCallback`, `arrowClickCallback`, `openCallback`, etc.) and reads their public `srcElement`/`value`/`selectedNodes`.
- DOM lifecycle: `mount()` tears down and rebuilds everything via `#initMount`/`#createTreeselect`; `destroy()` clears listeners and empties the container. Both wrappers (React/Vue) call `mount()` after mutating properties directly on the `Treeselect` instance rather than passing new full option objects — see `keysWithoutRender` lists in both wrappers for props that instead trigger a different path (value/options/id/iconElements changes require re-fetching data, not just a `mount()`).
- List positioning/direction (top vs bottom, `appendToBody` vs inline) is computed in `updateListPosition()` using `getBoundingClientRect`, driven by scroll/resize listeners registered only while the list is open.
- Selection semantics: three parallel value representations are tracked — `ungroupedValue` (leaf ids only, default), `groupedValue` (returns group ids when fully selected, `isGroupedValue` prop), and `allValue` (used when `isIndependentNodes` or `isSingleSelect`). `#updateInnerValues` picks which one becomes the public `value`.

`src/list/` internals:
- `listTypes.ts` defines `OptionsTreeMap` (a `Map<value, TreeItem>` — the flattened, mutable runtime representation of the nested `options` tree, built once by `getOptionsTreeMap` in `helpers/listOptionsHelper.ts`) and cached DOM references per node (`itemHtmlElement`, `checkboxHtmlElement`, `arrowItemHtmlElement`).
- `helpers/listCheckStateHelper.ts` — check/uncheck propagation logic (parent/child cascading, independent-nodes mode, disabled-branch handling).
- `helpers/listVisibilityStateHelper.ts` — search filtering, open/closed group state, and the `IntersectionObserver`-based "boosted rendering" mode (`isBoostedRendering`) for large trees.
- `helpers/domHelper.ts` — imperative DOM diffing/update (`updateDOM`) applied after any state change; the list does not use a virtual DOM, it mutates cached elements in place for performance.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dipson88/treeselectjs](https://github.com/dipson88/treeselectjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
