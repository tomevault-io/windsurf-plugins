---
trigger: always_on
description: This repository uses a pnpm + Nx TypeScript monorepo structure. The guidance below is written for agentic coding assistants that will run in this repo (linters, builders, test runners, and change authors). Follow these rules to produce consistent, reviewable changes.
---

# Agent Guide for Contributors and Automation

This repository uses a pnpm + Nx TypeScript monorepo structure. The guidance below is written for agentic coding assistants that will run in this repo (linters, builders, test runners, and change authors). Follow these rules to produce consistent, reviewable changes.

---

## Quick Commands

- Install dependencies: `pnpm i`
- Build all packages: `pnpm run build` (runs `build` + docs)
- Build affected packages: `pnpm run build:affected` or `pnpm nx affected -t build`
- Run the full workspace build (CI): `pnpm run build:ci`
- Run typedoc (docs): `pnpm --filter @tsparticles/website run typedoc:api`
- Start demo server: `cd demo/vanilla && pnpm start`

## Lint / Format / Commit hooks

- Format README + markdown: `pnpm run prettify:readme`
- Check formatting (CI): `pnpm run prettify:ci:readme`
- Run ESLint (package-local): `pnpm nx run <project>:lint` or `pnpm --filter <package> run lint`
- Husky hooks are enabled: local `git commit` runs hooks (`prepare` script in `package.json`). Do not bypass hooks.

## Tests

- Run all tests (workspace): `pnpm exec vitest` or `pnpm nx run-many -t test`
- Run package tests (example): `pnpm --filter @tsparticles/tests test`
- Run a single test file with Vitest:
  - From package: `pnpm --filter @tsparticles/tests test:particle` (predefined script)
  - Directly: `pnpm exec vitest run path/to/test/file.ts` (e.g. `utils/tests/src/tests/Particle.ts`)
  - With Nx (project-level): `pnpm nx test <project> --testFile=src/tests/Particle.ts` (use project-specific options)
- Run tests in watch UI: `pnpm --filter @tsparticles/tests test:ui` or `pnpm exec vitest --ui`
- Coverage: vitest with v8 provider configured in `utils/tests/vitest.config.ts` — CI exposes coverage artifacts under `utils/tests/coverage/`.

Notes on single-test execution: prefer package scripts that target a single file (many test packages include narrow scripts). If unavailable, run `pnpm exec vitest run <path>` from repo root.

---

## Build & Bundles

- Slim/full bundle build: `pnpm run build` (uses `nx run-many -t build`)
- Build for CI with module/UMD targets: `pnpm run build:ci`
- Bundle configs are under `bundles/*/webpack.config.js` and package-level `tsconfig.*.json` files.

---

## Code Style and Conventions

Follow the repository-wide style enforced by `@tsparticles/prettier-config` and `@tsparticles/eslint-config`.

General rules

- Language: TypeScript (>= 5.x) is the source language. Compiled outputs and bundles may be JavaScript.
- Formatting: Prettier (configured via `prettier` field in `package.json`). Run `pnpm run prettify:readme` for docs and `pnpm exec prettier --write .` for project-wide formatting when needed.
- Linting: ESLint with shared config `@tsparticles/eslint-config`. Run `npx eslint <path>` only when debugging lint issues.

Imports

- Group imports: 1) external packages, 2) workspace packages (internal), 3) relative imports; separate groups with a blank line.
- Prefer named imports for library exports; avoid default imports when the upstream uses named exports.
- Use absolute package imports for workspace packages (e.g. `@tsparticles/engine`) when available; otherwise use relative paths.

Types & API surfaces

- Prefer explicit types and interfaces for public package APIs. Avoid `any` on exported types.
- Use `unknown` instead of `any` when receiving untyped input and narrow immediately with type guards.
- Keep public option shapes documented and typed under `engine/src/Options/Interfaces/*`.

Naming

- Files exporting a class or main type should use matching PascalCase file names (e.g. `TiltUpdater.ts` exports `TiltUpdater`).
- Use PascalCase for classes & types, camelCase for functions/variables, UPPER_SNAKE_CASE only for constants that are truly constant across runs.

Error handling

- Throw `Error` for unrecoverable issues. Use typed Error subclasses for domain-specific errors when helpful.
- Prefer returning `undefined` for optional/absent results and document this in types (avoid using exceptions for control flow).
- Add defensive checks for external inputs (image URLs, user-provided options) and surface clear error messages.

Performance & hot paths

- Avoid per-frame allocations in hot loops; reuse objects or introduce pooling where it measurably improves throughput.
- Replace expensive operations on hot paths (e.g. `JSON.stringify` for memo keys) with bounded caches and efficient keying.

Global state

- Do not mutate `globalThis` unless strictly required; prefer opt-in bootstrap APIs that return instances.

Tests & fixtures

- Tests use Vitest + jsdom; canvas tests use `canvas` package and custom fixtures under `utils/tests/src/Fixture/`.
- Keep tests deterministic: avoid timing-based assertions; use mocked timers or explicit ticks where needed.

Commit messages

- Use Conventional Commits. Commitlint is enforced (`@commitlint/config-conventional`). Example: `fix(engine): handle NaN in velocity`.

PRs

- Provide clear description, testing notes, and update `CHANGELOG.md` if the change affects public APIs.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsparticles/tsparticles](https://github.com/tsparticles/tsparticles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
