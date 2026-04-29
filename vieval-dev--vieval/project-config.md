---
trigger: always_on
description: Concise but detailed reference for contributors working in the `vieval-dev/vieval` workspace. Improve code when you touch it; avoid one-off patterns.
---

# Vieval Agent Guide

Concise but detailed reference for contributors working in the `vieval-dev/vieval` workspace. Improve code when you touch it; avoid one-off patterns.

## Tech Stack (by surface)

- **Workspace**: `@vieval/root` (pnpm workspaces with `packages/**`).
- **Main package**: `packages/vieval` (`name: vieval`).
- **Runtime/tooling**: Node.js ESM, TypeScript, Vitest, tsdown, Turbo, pnpm.
- **Core libraries in use**: `vite`, `c12`, `meow`, `tinyglobby`, `es-toolkit`, `@xsai/generate-text`, `@xsai-ext/providers`, `@moeru/std`.
- **Lint/quality**: `moeru-lint` via root scripts (`lint`, `lint:fix`).

## Structure & Responsibilities

- **Root workspace**
  - Shared scripts and automation live in `package.json`, `turbo.json`, `pnpm-workspace.yaml`, and `.github/workflows`.
  - Lint and formatting are centralized through `moeru-lint` and `eslint.config.js`.
- **Main package (`packages/vieval`)**
  - `src/cli`: command entrypoint (`vieval`) and reporter implementations.
  - `src/config`: config definition/loading/types and model helpers.
  - `src/core/runner`: evaluation discovery, scheduling, execution, and aggregation.
  - `src/core/assertions`: assertion primitives/pipelines for eval outcomes.
  - `src/core/inference-executors`: model executor adapters, env handling, retry policy, provider bindings.
  - `src/dsl`: task and registry APIs.
  - `src/plugins/chat-models`: plugin exports for chat-model integrations.
  - `src/testing`: testing helpers and expect extensions.
- **Packaging/build**
  - Package build is `tsdown` (`packages/vieval/tsdown.config.ts`).
  - Typecheck/test configs are local to the package (`packages/vieval/tsconfig.json`, `packages/vieval/vitest.config.ts`).

## Key Path Index (what lives where)

- Root:
  - `package.json`: workspace scripts (`build`, `build:packages`, `typecheck`, `test:run`, `lint`, `lint:fix`).
  - `pnpm-workspace.yaml`: workspace and catalog configuration.
  - `turbo.json`: task orchestration for package builds.
  - `vitest.config.ts`: root Vitest setup.
  - `eslint.config.js`: linting rules.
  - `.github/workflows`: CI pipeline definitions.
- Main package:
  - `packages/vieval/package.json`: package exports, `vieval` CLI bin, package-level scripts.
  - `packages/vieval/src/index.ts`: package public entrypoint.
  - `packages/vieval/src/cli/index.ts`: CLI entrypoint.
  - `packages/vieval/src/core/runner/`: runner pipeline.
  - `packages/vieval/src/core/assertions/`: assertion APIs.
  - `packages/vieval/src/core/inference-executors/`: inference execution internals.
  - `packages/vieval/src/config/`: config API and model definitions.
  - `packages/vieval/src/dsl/`: task/registry DSL.
  - `packages/vieval/src/testing/`: test helpers and expect extensions.
  - `packages/vieval/tests/projects/`: integration-like example projects for eval flows.
  - `packages/vieval/tsdown.config.ts`: package bundling config.
  - `packages/vieval/vitest.config.ts`: package-level Vitest config.

## Commands (pnpm with filters)

> This repo currently contains one package: `vieval` (`packages/vieval`). Use either root scripts or `pnpm -F vieval ...`.

- **Typecheck**
  - Root: `pnpm typecheck`
  - Package: `pnpm -F vieval typecheck`
- **Unit tests (Vitest)**
  - Targeted: `pnpm exec vitest run <path/to/file>`
    e.g. `pnpm exec vitest run packages/vieval/src/core/runner/run.test.ts`
  - Package: `pnpm -F vieval exec vitest run`
  - Root: `pnpm test:run`
- **Lint**
  - `pnpm lint` and `pnpm lint:fix`
  - Formatting/fixes are handled via `moeru-lint` (ESLint-based).
- **Build**
  - Root: `pnpm build` or `pnpm build:packages`
  - Package: `pnpm -F vieval build`

## Development Practices

- Favor clear module boundaries; shared logic goes in `packages/`.
- Keep runtime entrypoints lean; move heavy logic into services/modules.
- Prefer functional patterns + DI (`injeca`) for testability.
- Use Valibot for schema validation; keep schemas close to their consumers.
- Use Eventa (`@moeru/eventa`) for structured IPC/RPC contracts where needed.
- Use `errorMessageFrom(error)` from `@moeru/std` to extract error messages instead of manual patterns like `error instanceof Error ? error.message : String(error)`. Pair with `?? 'fallback'` when a default is needed.
- Do not add backward-compatibility guards. If extended support is required, write refactor docs and spin up another Codex or Claude Code instance via shell command to complete the implementation with clear instructions and the expected post-refactor shape.
- If the refactor scope is small, do a progressive refactor step by step.
- When modifying code, always check for opportunities to do small, minimal progressive refactors alongside the change.

## Styling & Components

- Prefer Vue v-bind class arrays for readability when working with UnoCSS & tailwindcss: do `:class="['px-2 py-1','flex items-center','bg-white/50 dark:bg-black/50']"`, don't do `class="px-2 py-1 flex items-center bg-white/50 dark:bg-black/50"`, don't do `px="2" py="1" flex="~ items-center" bg="white/50 dark:black/50"`; avoid long inline `class=""`. Refactor legacy when you touch it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vieval-dev/vieval](https://github.com/vieval-dev/vieval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
