---
trigger: always_on
description: - pnpm workspace, TypeScript, Vitest, ESLint.
---

# `alint` Agent Guide

- pnpm workspace, TypeScript, Vitest, ESLint.
- Components:
  - `packages/cli`: CLI entrypoint, argument parsing, setup/config commands, reporter output, and handoff into core.
  - `packages/config`: project config loading plus global/local setup TOML paths, parsing, merging, and writing.
  - `packages/core`: public rule/plugin DSL, run engine, source extraction/runtime, model resolution, diagnostics, cache, and usage tracking.
  - `packages/plugin-example`: example rule plugin that exercises the public SDK and model-backed rule flow.

## Structure & Responsibilities

- Build pipeline refs: `.github/workflows`; lint rules in `eslint.config.ts`.

## Commands (pnpm with filters)

> Use pnpm workspace filters to scope tasks. Examples below are generic; replace the filter with the target workspace name (e.g. `@alint-js/core`, etc.).

- **Typecheck**
  - `pnpm -F <package.json name> typecheck`
  - Example: `pnpm -F @alint-js/cli typecheck` (runs `tsc`).
- **Unit tests (Vitest)**
  - Targeted: `pnpm exec vitest run <path/to/file>`
    e.g. `pnpm exec vitest run packages/cli/src/cli/cli.test.ts` (not recommended as we prefer to run entire package test instead of file)
  - Workspace: `pnpm -F <package.json name> exec vitest run`
    e.g. `pnpm -F @alint-js/cli exec vitest run`
  - Root `pnpm test:run`: runs all tests across registered projects. If no tests are found, check `vitest.config.ts` include patterns.
  - Root `vitest.config.ts` includes `packages/cli` and other projects; each app/package can have its own `vitest.config`.
- **Lint**
  - `pnpm lint` and `pnpm lint:fix`
  - Formatting is handled via ESLint; `pnpm lint:fix` applies formatting.
- **Build**
  - `pnpm -F <package.json name> build`
  - Example: `pnpm -F @alint-js/cli build` (typecheck + electron-vite build).

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

## Testing Practices

- Vitest per project; keep runs targeted for speed.
- For any investigated bug or issue, try to reproduce it first with a test-only reproduction before changing production code. Prefer a unit test; if that is not possible, use the smallest higher-level automated test that can still reproduce the problem.
- When an issue reproduction test is possible, include the tracker identifier in the test case name:
  - GitHub issues: include `Issue #<number>`
  - Internal bugs tracked in Linear: include the Linear issue key
- Add the actual report link as a comment directly above the regression test:
  - GitHub issue URL for GitHub reports
  - Discord message or thread URL for IM reports
  - Linear issue URL for internal bugs
- Mock IPC/services with `vi.fn`/`vi.mock`; do not rely on real Electron runtime.
- For external providers/services, add both mock-based tests and integration-style tests (with env guards) when feasible. You can mock imports with Vitest.
- Grow component/e2e coverage progressively (Vitest browser env where possible). Use `expect` and assert mock calls/params.
- When writing tests, prefer line-by-line `expect` or assertion statements.
- Avoid writing tests for impossible runtime states, such as `expect` against constants that never change, or asserting object mutations that can only happen inside the same Vitest case setup.
- Avoid mocking `globalThis` or built-in modules by directly using `Object.defineProperty(...)`. If needed, use `node:worker_threads` to load another worker and simulate that situation, or build a mini CLI to reproduce and verify behavior. For DOM and Web Platform APIs, prefer Vitest browser mode instead of hard-mocking platform internals. If tests already use those patterns, progressively refactor them.
- Do not use Vitest mocks, hoisting, dynamic imports, `as unknown as`, or test-only alternate import paths to maliciously bypass real import problems. If a test cannot import a module, investigate the actual compile/runtime boundary: package exports, side effects, mixed Node/browser type dependencies, circular imports, and whether the public module shape is wrong. Fix the boundary instead of hiding the failure in the test.

## TypeScript / IPC / Tools

- Keep JSON Schemas provider-compliant (explicit `type: object`, required fields; avoid unbounded records).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moeru-ai/alint](https://github.com/moeru-ai/alint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
