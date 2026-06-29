---
trigger: always_on
description: This repository contains `retry-axios`, a TypeScript library that adds configurable retry behavior to Axios requests through interceptors.
---

# AGENTS.md

## Purpose

This repository contains `retry-axios`, a TypeScript library that adds configurable retry behavior to Axios requests through interceptors.

Use this file as the default contributor and coding-agent playbook for the repo. Prefer repo-specific guidance here over generic habits.

## Project Snapshot

- Language: TypeScript
- Package manager: `npm`
- Runtime support: Node.js `>=20`
- Module outputs: ESM and CommonJS
- Source of truth: `src/`
- Tests: `test/` with Vitest
- Formatting and linting: Biome
- Release automation: `release-please`

## Repository Layout

- `src/index.ts`: main library implementation
- `test/index.ts`: primary behavior tests
- `test/cjs-import.test.cjs`: CommonJS import compatibility coverage
- `examples/`: usage examples and sample integration
- `build/`: generated artifacts from compilation; do not treat as hand-edited source
- `.github/workflows/ci.yaml`: pull request and main-branch CI checks
- `.github/workflows/release.yaml`: automated release and publish workflow
- `CHANGELOG.md`: generated release history

## Environment Expectations

- Use `npm` for installs, scripts, and lockfile updates
- Develop against Node.js 20 or newer
- Keep `package-lock.json` in sync when dependency changes are intentional
- Axios is a peer dependency; local tests rely on the repo's dev dependency copy

## Important Scripts

- `npm run lint`: Biome lint/format validation
- `npm run fix`: Biome autofix
- `npm run typecheck`: strict TypeScript validation with `tsconfig.typecheck.json`
- `npm run compile`: build ESM and CJS outputs
- `npm run compile:esm`: TypeScript compile to `build/`
- `npm run compile:cjs`: bundle CommonJS output with esbuild
- `npm test`: run compile first, then execute Vitest with coverage
- `npm run test:watch`: watch mode for tests
- `npm run license-check`: local license audit

## Source Control Rules

- Never commit directly to `main`
- Start work from a branch
- Keep changes focused; avoid mixing refactors, dependency churn, and behavior changes unless required
- Do not manually edit generated release metadata unless the task explicitly calls for it

## Generated Files

- `build/` is generated output and is not tracked in git in this repository
- Make source changes in `src/`, not in `build/`
- Run `npm run compile` locally when you need to validate packaging behavior
- Do not add generated `build/` artifacts to commits unless project policy changes in the future

## Coding Guidelines

- Preserve the public API unless the task explicitly requires an API change
- Follow the existing code style and naming patterns in `src/index.ts`
- Keep the implementation dependency-light and package-friendly
- Prefer small, explicit changes over broad rewrites
- Maintain compatibility with strict TypeScript settings
- Avoid introducing new tooling when existing Biome, TypeScript, and Vitest workflows already cover the need

## Testing Expectations

Run the following for most code changes:

1. `npm run lint`
2. `npm run typecheck`
3. `npm test`

Additional guidance:

- Add or update tests in `test/` whenever runtime behavior changes
- If changing module/export behavior, verify both the TypeScript tests and `test/cjs-import.test.cjs`
- If changing docs or examples only, lighter validation may be enough, but do not skip checks if behavior also changed

## CI Expectations

Pull requests are validated by `.github/workflows/ci.yaml`, which runs:

- Link checking
- Tests on Node.js 20, 22, and 24
- Lint
- Typecheck
- License check

Before opening a PR, contributors should aim to pass the local equivalents so CI is mostly a confirmation step.

## Conventional Commits

This repository follows Conventional Commits, and commit messages matter because releases and changelog entries are generated from commit history.

Use this format:

```text
type(scope): short summary
```

Examples:

- `feat: add support for retry-after edge case`
- `fix: preserve retry config on cloned requests`
- `docs: clarify TypeScript import usage`
- `test: cover CommonJS import path`
- `chore(deps): update vitest to latest compatible version`

Common types to use here:

- `feat`: new user-facing functionality
- `fix`: bug fix
- `docs`: documentation-only change
- `test`: test-only change
- `chore`: maintenance, tooling, or non-user-facing work
- `refactor`: internal code restructuring without intended behavior change

Additional commit guidance:

- Keep the subject short and imperative
- Use lowercase types
- Add a scope when it makes the change clearer, but do not force one
- Use `!` or a `BREAKING CHANGE:` footer only for intentional breaking changes
- Avoid vague messages like `updates`, `fix stuff`, or `changes`

## Pull Request Instructions

Follow this workflow unless the maintainer asks for something different.

### 1. Sync with `main`

```sh
git checkout main
git pull --ff-only origin main
```

### 2. Create a branch

Use a short descriptive branch name:

```sh
git checkout -b fix/retry-after-handling
```

### 3. Make changes

- Edit source under `src/` and tests under `test/`
- Update docs/examples if user-facing behavior or usage changes

### 4. Validate locally

```sh
npm run lint
npm run typecheck
npm test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JustinBeckwith/retry-axios](https://github.com/JustinBeckwith/retry-axios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
