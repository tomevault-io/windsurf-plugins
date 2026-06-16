---
trigger: always_on
description: This file provides guidance to AI coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working in this repository.

## Core rules

- Use `yarn`, not `npm`, for repo commands. This repo uses Yarn workspaces and Yarn 4.
- Run commands from the repo root unless a command explicitly says to run from a workspace.
- Never run bare `tsc`; use `yarn typecheck` from the repo root.
- Prefer targeted checks first. Avoid repo-wide test or e2e runs unless the change needs them.
- Keep changes scoped to the request and the affected package. Do not refactor unrelated code.
- Respect existing worktree changes. Do not revert user changes unless explicitly asked.
- Prefer editing existing files over creating new files. Do not add new documentation files unless requested.
- Use sentence case for headings, titles, labels, and documentation text.

## Repo overview

This is the tldraw monorepo, an infinite canvas SDK for React applications. It is organized with Yarn workspaces.

Core packages:

- `packages/editor` - foundational infinite canvas editor with no default shapes, tools, or UI
- `packages/tldraw` - complete SDK with default UI, shapes, tools, and interactions
- `packages/store` - reactive client-side database, persistence, and migrations
- `packages/tlschema` - shape, binding, and record type definitions and validators
- `packages/state` - reactive signals library
- `packages/sync` and `packages/sync-core` - multiplayer sync packages
- `packages/utils` and `packages/validate` - shared utilities and validation helpers
- `packages/assets` - icons, fonts, translations, and bundled assets

Apps and examples:

- `apps/examples` - SDK examples and demos; the main place for example development
- `apps/docs` - documentation site at tldraw.dev
- `apps/dotcom` - tldraw.com app and workers
- `apps/vscode` - VS Code extension
- `templates` - starter templates for supported frameworks

## Setup

Requires Node `^20.0.0`. Enable Corepack before installing dependencies:

```bash
npm i -g corepack && yarn
```

## Common commands

Development:

- `yarn dev` - start the examples app at localhost:5420
- `yarn dev-app` - start the tldraw.com client app
- `yarn dev-docs` - start the docs site
- `yarn dev-vscode` - start VS Code extension development
- `yarn dev-template <template name>` - run a template

Build:

- `yarn build` - build all changed packages incrementally
- `yarn build-package` - build SDK packages only
- `yarn build-app` - build the tldraw.com client app
- `yarn build-docs` - build the docs site

Testing:

- `yarn test` in a workspace - run tests in watch mode
- `yarn test run` in a workspace - run tests once
- `yarn test run --grep "pattern"` in a workspace - run matching tests
- `yarn vitest` - run all tests across the repo; slow, avoid unless necessary
- `yarn e2e` - run examples e2e tests
- `yarn e2e-dotcom` - run tldraw.com e2e tests

Code quality:

- `yarn lint` - lint the package or workspace
- `yarn lint-current` - lint changed files
- `yarn typecheck` - type check all packages and refresh assets
- `yarn format` - format the repo
- `yarn format-current` - format changed files
- `yarn api-check` - validate public API reports

## Validation workflow

- For narrow package changes, run the relevant workspace test first, for example `cd packages/tldraw && yarn test run --grep "SelectTool"`.
- For changes that affect shared types, migrations, editor behavior, or cross-package contracts, run `yarn typecheck` from the repo root.
- For public API changes, run `yarn api-check` and include intentional API report updates.
- For asset changes, run `yarn refresh-assets` or `yarn typecheck` so generated assets stay current.
- For docs changes, run the narrow docs checks or docs build only when the change affects generated content, MDX behavior, or site structure.
- For e2e behavior changes, run the smallest relevant e2e suite and update snapshots only when behavior intentionally changed.

## Architecture notes

Reactive state:

- State is managed through `@tldraw/state` signals (`Atom`, `Computed`, and related primitives).
- Editor state is observable and dependency-tracked. Avoid bypassing existing reactive patterns.

Shapes:

- Shape behavior lives in `ShapeUtil` classes.
- Shape utils define geometry, rendering, handles, interactions, and SVG/export behavior.
- Add custom shape behavior through the established ShapeUtil patterns rather than one-off editor patches.

Tools:

- Tools are `StateNode` state machines.
- Complex tools use child states for pointer, keyboard, tick, and transition behavior.
- Keep interaction logic close to the tool state that owns it.

Bindings:

- Shape relationships use binding records and `BindingUtil` classes.
- Arrows and other connected shapes should update through binding utilities, not ad hoc shape mutation.

Store and schema:

- Store changes should respect migrations, validators, and schema versioning.
- Schema-affecting changes usually need updates in `packages/tlschema` and focused migration tests.

## Where to work

- Use `packages/editor` for core editor primitives, geometry, managers, and UI-free behavior.
- Use `packages/tldraw` for default shapes, default tools, UI, and integration tests that need the full SDK.
- Use `apps/examples` for runnable SDK examples and demonstrations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whiteg2030-alt/wuxianhuabu](https://github.com/whiteg2030-alt/wuxianhuabu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
