---
trigger: always_on
description: This file gives guidance to AI and automation agents working in this repository. Before changing code, read the root `README.md` and the relevant package `package.json`/`project.json`, then follow the existing local patterns.
---

# AGENTS.md

This file gives guidance to AI and automation agents working in this repository. Before changing code, read the root `README.md` and the relevant package `package.json`/`project.json`, then follow the existing local patterns.

## Project Overview

AnkiEco is a monorepo for building cross-platform Anki experiences, including documentation, publishable packages, extensions, and reference templates.

- `apps/docs`: VitePress documentation site with content split between `src/en` and `src/zh`.
- `packages/dev-ui`: Development/debug UI built with Preact.
- `packages/extensions`: Web component extensions published as `@anki-eco/extensions`, including CardMotion, Tldraw, and XMarkdown.
- `packages/kit`: Runtime helpers for templates and extensions, with vanilla, React, and Vue entry points.
- `packages/packager`: `@anki-eco/packager` CLI, with TypeScript entry points and Python packaging logic under `src/python`.
- `packages/shared`: Shared types, utilities, and `packager-schema.json`.
- `packages/vite-plugin`: AnkiEco Vite plugin. Its package entry points live directly in `src/*.js` and `src/*.d.ts`.
- `templates/classic`: Main Classic template project with build scripts, React/Preact template source, tests, and Python packaging scripts.
- `templates/example-react` and `templates/example-vue`: Example templates using `@anki-eco/kit` and `@anki-eco/vite-plugin`.

## Environment And Package Management

- Use Bun as the package manager, script runner, and runtime.
- Classic template and packager builds require `uv`; CI uses `uv 0.4.27`.
- Install dependencies with `bun install`.
- Do not switch to npm, yarn, or pnpm, and do not hand-edit `bun.lock`; dependency changes should go through `bun`.

## Common Commands

Run these from the repository root:

- `bun run lint`: Run `oxlint`.
- `bun run lint:fix`: Apply available lint fixes.
- `bun run fmt`: Format with `oxfmt`.
- `bun run fmt:check`: Check formatting.
- `bunx nx show projects`: List Nx project names.
- `bunx nx affected -t lint test build typecheck package fmt:check`: Run a CI-like affected-project check.
- `bunx nx run @anki-eco/docs:dev`: Start the docs site.
- `bunx nx run @anki-eco/docs:build`: Build the docs site.
- `bunx nx run @anki-eco/classic-templates:dev -- mcq --locale=zh --field=markdown`: Develop a specific Classic template.
- `bunx nx run @anki-eco/classic-templates:build`: Build the Classic templates.
- `bunx nx run @anki-eco/classic-templates:test`: Run Classic template tests.
- `bunx nx run @anki-eco/classic-templates:package`: Package Classic templates via `uv run --frozen build/package.py`.
- `bunx nx run @anki-eco/packager:build`: Build the packager TypeScript output and Python executable.
- `bunx nx run @anki-eco/packager:test`: Validate packager output with `test_data`.

You can also run workspace scripts directly, for example:

- `bun run --filter @anki-eco/docs dev`
- `bun run --filter @anki-eco/classic-templates test`

## Validation Strategy

- After completing any change, run `bun run lint` and `bun run fmt`.
- For small TypeScript/React/Vue changes, run the relevant project `typecheck` or `test`, plus root `bun run lint`/`bun run fmt:check` when appropriate for the risk.
- For Classic template behavior changes, prefer adding or updating Vitest coverage under `templates/classic/tests`, then run `bunx nx run @anki-eco/classic-templates:test`.
- For Classic build or packaging changes, run `build`; if `.apkg` output or Python scripts are involved, run `package`.
- For packager changes, run `bunx nx run @anki-eco/packager:build`; run `test` when packaging behavior may have changed.
- For documentation changes, run `bunx nx run @anki-eco/docs:build`, and check whether both English and Chinese docs need updates.
- For publishable package entry point or export changes, check the relevant `package.json` `exports`, `types`, `files`, and tsconfig references.

If local environment issues, runtime, or missing dependencies prevent validation, state exactly which commands were not run and why in the final response.

## Code Style

- TypeScript is strict. Root `tsconfig.base.json` enables `strict`, `noImplicitReturns`, `noUnusedLocals`, `noFallthroughCasesInSwitch`, and related checks.
- Formatting uses `oxfmt` with single quotes. Do not introduce Prettier or ESLint config as a replacement for the existing tools.
- Linting uses `oxlint` with eslint/typescript/unicorn/react/vue/vitest/import plugins and type-aware checks.
- Prefer existing utilities, stores, hooks, and package boundaries. Avoid creating new global abstractions for local needs.
- Keep ESM style. If a package already uses `.js` source files, preserve that language and export style.
- Avoid committing generated output and caches such as `dist`, `coverage`, `.nx/cache`, and `.nx/workspace-data`, unless the task explicitly requires release artifacts.

## Classic Template Notes

- Main source is in `templates/classic/src`:
  - `entries`: Template entry points.
  - `features`: Interactive features such as `cloze`, `markdown`, `ordering`, `tf`, and `tools`.
  - `store`: Jotai/state configuration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ikkz/anki-eco](https://github.com/ikkz/anki-eco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
