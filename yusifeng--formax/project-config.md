---
trigger: always_on
description: - Keep high-frequency operational guidance here (review workflow/profile, commonly used commands/scripts, commit workflow, and day-to-day guardrails).
---

# Repository Guidelines

## AGENTS Scope

- Keep high-frequency operational guidance here (review workflow/profile, commonly used commands/scripts, commit workflow, and day-to-day guardrails).
- Keep canonical product/runtime semantics in `docs/contracts/*` and related canonical docs.
- Keep code navigation in `CODEMAP.md` ("where to change what"), with AGENTS linking to it instead of duplicating path-level maps.

## Project Structure & Module Organization

- `packages/core/src/` contains TypeScript source.
  - `entrypoints/` CLI/runtime entrypoints (for example `cli.tsx`).
  - `runtime/cli/` argument parsing + command dispatch for the CLI wrapper.
  - `runtime/bootstrap/` REPL bootstrap + runtime assembly wiring.
  - `core/` productized app core (config resolution, setup flows, boundaries checks).
  - `tui/` Ink “wizard” UIs (first-run setup, selectors, forms).
  - `adapters/` filesystem + setup adapters (config files, setup persistence).
  - `screens/` Ink screens; `components/` reusable UI.
  - `tools/` tool registry, modules, handlers, presenters, runtime managers.
  - `streaming/` Anthropic streaming client and parsers.
  - `features/subagents/` registry/runner for sub-agent tools.
  - `prompts/`, `config/`, `services/`, `shared/utils/` supporting code.
- `packages/` contains companion workspaces. Package-local implementation details belong to each package `AGENTS.md`.
- Tests live next to source as `*.test.ts`/`*.test.tsx`.
- `docs/` holds architecture notes and guides; `plans/` captures refactor plans.
- `proxy/` contains proxy/logger scripts plus traffic/log artifacts used for parity/reference during development.
- `CODEMAP.md` is the “where to change what” index (entrypoints, main loop, tools, plan mode, sub-tasks).

## Build, Test, and Development Commands

- `bun install` installs dependencies.
- `bun run dev` runs the CLI via `tsx` (entry: `packages/core/src/entrypoints/cli.tsx`).
- `bun run app-server:bridge -- --host 127.0.0.1 --port 3777` runs the app-server WebSocket bridge for web client development.
- `bun run build` bundles the CLI to `dist/cli.js` (requires Bun).
- `bun run type-check` runs TypeScript checks + boundary checks (`core` + `ui`).
- `bun run check:root-script-governance` enforces root `package.json` script governance (orchestration-only, no feature alias uplift).
- `bun run ui:boundaries` runs UI boundary checks (guards `packages/core/src/tui/`, `packages/core/src/screens/`, `packages/core/src/components/` from importing forbidden layers).
- `bun run test` runs `vitest run`; `bun run test:watch` runs Vitest watch.
- Single test: `bun run test -- packages/core/src/tools/registry.test.ts`.
- Package-specific commands (web/desktop/shared/semantics) live in each package `AGENTS.md`.

## Coding Style & Naming Conventions

- TypeScript ESM (`"type": "module"`, bundler module resolution).
- Tool modules follow `packages/core/src/tools/modules/<name>/{index,handler,presenter}.ts(x)` and `createXToolModule` factory naming.
- Root script governance: keep feature/package workflows in owning package scripts; root scripts are orchestration-level only (see `docs/contracts/root-script-governance-contract.md`).
- **GLOBAL CLARIFICATION RULE (MANDATORY)**: In ANY task, if user intent is ambiguous (UI, behavior, scope, risk, tradeoff, or acceptance criteria), you MUST ask the user BEFORE making directional choices. DO NOT infer beyond explicit requirements.
- **QUESTION TOOL RULE (MANDATORY)**: When clarification is needed, you MUST use the user-input question tool first (e.g. `request_user_input`) when available. If that tool is unavailable in the current mode, you MUST ask the user directly in chat instead of guessing.

## Testing Guidelines

- Framework: Vitest; Ink UI tests use `ink-testing-library`.
- Property-based tests use `fast-check` where appropriate.
- Keep tests colocated with source and use `*.test.ts`/`*.test.tsx`.
- **Coverage mindset**: Prioritize adding/strengthening tests when behavior is user-visible or stability-critical (tools, permissions, hooks, REPL input, UI flows). Avoid “happy-path only” tests—cover edge cases and regressions you’ve already seen.
- **Refactor safety**: Before refactoring, add/extend tests to lock current behavior. Do not rely on “tests pass” if manual behavior regresses.
- **Code review (mandatory in loops)**: After tests pass, run review using the **Review Profile (Single Source of Truth)** below before committing; fix all high/medium findings (and any low-risk issues that are clearly correct and low-churn).
- **REPL semantics pre-review gate**: For `packages/core/src/features/repl/**` semantic-flow changes, run `bun run test:repl-semantic-gate` before `codex review`.

### Review Profile (Single Source of Truth)

- Review command: `codex review --uncommitted -c model="gpt-5.3-codex" -c model_reasoning_effort="medium"`
- Tool-call timeout for review: `timeout_ms >= 1200000`
- Review output artifact path (preferred): `.tmp/codex-review-result/`
- Preferred local wrapper when running review manually or from agents:
  - `mkdir -p .tmp/codex-review-result`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yusifeng/formax](https://github.com/yusifeng/formax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
