---
trigger: always_on
description: This file provides guidance to AI coding agents working with this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with this repository.

## Repository purpose

`react-d3-tree` is a React component that renders hierarchical data (org charts, family trees, file directories) as an interactive SVG tree graph, built on D3's `tree` layout from `d3-hierarchy`. It ships to npm as a library consumed by other React apps. The `demo/` directory holds the playground, a Vite app in the pnpm workspace that consumes the library through `workspace:*` and deploys to GitHub Pages through the `Pages` workflow.

## Backwards compatibility

This library is published to npm, so anything a consuming app relies on is a contract. Keep minor and patch releases backwards compatible — a developer bumping within the same major version must not have their build, types, or app break. Introduce a breaking change only when the task is explicitly about one, and land it in a major version.

Beyond obvious source-level API changes, a change is breaking if it affects any of these:

- Public API: renaming, removing, or changing the behavior of the `src/index.ts` exports (`Tree`, its props, its defaults, or the exported types).
- Peer dependencies: narrowing the supported `react`/`react-dom` range (16.x–19.x) or adding a new required peer dependency.
- Build output: changing which files the `exports` map or the `main`, `module`, and `types` fields resolve to for a consumer that works today, dropping a module format, or raising the compile target (CJS `es5`, ESM `es6`) so runtimes or bundlers that work today stop working. Restructuring `exports` is fine when every existing consumer keeps resolving the same runtime file and equivalent types; `pnpm check:package` and the consumer type-checks in `pnpm test:smoke` are the evidence.
- Shipped types: raising the minimum TypeScript version the `.d.ts` files need, or changing emitted types so existing consumer code stops type-checking.

When a change might break consumers, don't assume either way. Research and validate it: build the package before and after and compare the output, run the smoke test and any package checks the repo has, and test the specific consumer setup the change could affect (module format, resolution mode, TypeScript version). Record what you verified and what stays unverified. If the evidence still leaves a judgement call, for example a fix that changes what some consumers see, give the maintainer the evidence and let them decide; don't classify the change as breaking or safe on an untested assumption.

## Architecture

The library source lives in `src/`. Everything else supports building, testing, docs, or the demo.

- `src/index.ts` — public API entry point. Exports `Tree` (default and named) and re-exports the public types.
- `src/Tree/index.tsx` — the `Tree` component, a class component that owns layout state, zoom, pan, and collapse/expand. Computes the layout with `d3-hierarchy` and wires zoom with `d3-zoom`/`d3-selection`.
- `src/Tree/TransitionGroupWrapper.tsx` — animation wrapper around `@bkrem/react-transition-group`.
- `src/Tree/types.ts` — `Tree` prop and callback types.
- `src/Node/index.tsx` — renders a single node; `src/Node/DefaultNodeElement.tsx` is the default node renderer used when no custom renderer is supplied.
- `src/Link/index.tsx` — renders the path between two nodes; supports the built-in `pathFunc` variants and a caller-supplied function.
- `src/types/common.ts` — shared data types (`RawNodeDatum`, `TreeNodeDatum`, `Point`, event handler types).
- `src/generateId.ts` — generates the v4 UUIDs that `Tree` uses for its SVG and group class references and for node IDs. Not part of the public API.
- `src/globalCss.ts` — injected base styles.

The build emits four artifacts under `lib/`: CommonJS (`lib/cjs`), ES modules (`lib/esm`), type declarations (`lib/types`), and a copy of the declarations for CommonJS consumers (`lib/types-cjs`). Because the root `package.json` sets `"type": "module"`, `scripts/mark-cjs.js` writes a `package.json` with `"type": "commonjs"` into `lib/cjs` and `lib/types-cjs`; without the second marker TypeScript reads the declarations as ESM and rejects them from a CommonJS file under `node16` resolution. The `package.json` `exports` map lists `types` before `default` under both the `import` and the `require` condition.

## Tech stack

- pnpm 12 as the package manager, pinned through `packageManager` in `package.json`. Development needs Node 22.22.2 or later, or 24.15 or later: the highest `engines.node` floor among the dev dependencies (`jsdom`). pnpm doesn't enforce engine ranges by default, so an older Node installs with no error but runs tooling outside its supported range. When a dev dependency raises its floor, update this line and the README. The `demo/` app is a workspace package (`pnpm-workspace.yaml`), so one lockfile covers both and `pnpm install` at the root installs it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bkrem/react-d3-tree](https://github.com/bkrem/react-d3-tree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
