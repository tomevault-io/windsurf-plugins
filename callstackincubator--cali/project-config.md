---
trigger: always_on
description: Minimal operating guide for AI coding agents in this repo.
---

# AGENTS.md

Minimal operating guide for AI coding agents in this repo.

## First 60 Seconds

- Classify the task:
  - Info-only: do not edit code or run checks unless needed.
  - Code change: make the smallest scoped edit and run the lightest relevant validation.
- Read at most 4 files first:
  - the owning command module
  - one role module
  - one shared runtime file
  - one relevant doc file if the CLI or context contract changes
- Define concrete success criteria before editing.
- Prefer the shared runtime contracts over command-local improvisation.

## Repo Shape

- `packages/cali`: standalone CLI role platform
- `packages/tools`: reusable Cali tools for other runtimes

## Cali Runtime Shape

The `cali` package is now a small role platform.

- CLI entry:
  - `packages/cali/src/cli.ts`
  - `packages/cali/src/cli/app.ts`
  - `packages/cali/src/cli/*.ts`
- Command orchestration:
  - `packages/cali/src/commands/*.ts`
- Shared runtime:
  - `packages/cali/src/runtime/types.ts`
  - `packages/cali/src/runtime/context.ts`
  - `packages/cali/src/runtime/tool-packs.ts`
  - `packages/cali/src/runtime/tool-loop-role.ts`
  - `packages/cali/src/runtime/publishers.ts`
  - `packages/cali/src/runtime/mobile.ts`
- Config:
  - `packages/cali/src/config/schema.ts`
  - `packages/cali/src/config/load.ts`
- Roles:
  - `packages/cali/src/roles/*.ts`
- Tool packs:
  - `packages/cali/src/tools/*.ts`
- Reports:
  - `packages/cali/src/report/types.ts`
  - `packages/cali/src/report/render.ts`
  - `packages/cali/src/report/publishers/*.ts`

## Public Commands

Implemented first-class commands:

- `qa`
- `review`
- `perf-review`
- `dev`

Current maturity:

- `qa`: ship-ready
- `review`: experimental
- `perf-review`: experimental
- `dev`: experimental

`publish` is intentionally not implemented. Release automation belongs in CI or in `dev`-driven pipeline work, not as an open-ended agent command.

## Core Contracts

### Local Mode

Use `--local android|ios` for local mobile runs.

CI metadata is detected automatically in GitHub Actions and EAS. Use `--ci github-actions|eas` only when you need to override detection.

### Context

All commands use one shared `cali-context.json` contract.

Keep the shared context focused on:

- `workspaceRoot`
- `repository`
- `task`
- `pullRequest`
- `mobile`
- `build`
- `output`
- role-specific optional sections:
  - `qa`
  - `review`
  - `perfReview`
  - `dev`

If a new workflow needs more data, extend the shared context schema in `packages/cali/src/runtime/context.ts` instead of adding a new workflow-specific loader.

### Tool Packs

Built-in pack ids:

- `skills`
- `agent-device`
- `repo-read`
- `repo-write`
- `react-devtools`

Required skill guidance should be preloaded through the tool-pack registry when a pack depends on a skill workflow. Do not push that responsibility into individual prompts by hand.

Required role skills are Cali-managed:

- Cali auto-installs missing required skills into `~/.cali/skills`
- if that is unavailable, Cali falls back to `./.cali/skills`
- local CLIs are still user-managed; do not blur skill bootstrap with CLI installation

## Command Guidance

### `qa`

- Bootstrap stays outside the role in the command module.
- The role inspects the app and writes a structured QA report.
- Use `--local android|ios` for local runs.
- In GitHub Actions and EAS, CI provider detection is automatic; `--ci` is only an override.
- Requires `agent-device` on `PATH`.
- Mobile runs use a unique per-run `agent-device` session. Do not reuse ambient sessions.
- Local runs are convenience-first: try `open --relaunch` before reinstalling.
- Local mobile runs can infer the app id from the artifact. Do not require `--app-id` unless inference fails.
- If `--device` is omitted, reuse the single booted local target when exactly one exists; otherwise fail clearly.
- Acceptance criteria resolve in this order:
  - `context.qa.acceptanceCriteria`
  - `context.pullRequest.body`
  - `context.task.body`
  - additive CLI prompt

### `review`

- No code changes.
- In GitHub Actions and EAS, CI-derived repository and PR metadata is detected automatically. Use `--ci` only to override detection.
- Findings first.
- Prefer repository/diff evidence over generic advice.

### `perf-review`

- Uses both `agent-device` and `react-devtools`.
- Use `--local android|ios` for local runs.
- In GitHub Actions and EAS, CI provider detection is automatic; `--ci` is only an override.
- Requires `agent-device` and `agent-react-devtools` on `PATH`.
- Focus on runtime evidence, not speculative optimizations.

### `dev`

- Smallest code change that solves the task.
- In GitHub Actions and EAS, CI-derived repository and PR metadata is detected automatically. Use `--ci` only to override detection.
- Repository tools rely on `git`, `rg`, and `zsh` being available.
- Respect `context.dev.writePolicy` and `context.dev.pushPolicy`.

## Validation

- For `packages/cali` TypeScript changes:
  - `bunx tsc --noEmit -p packages/cali/tsconfig.json`
- For `packages/tools` TypeScript changes:
  - `bunx tsc --noEmit -p packages/tools/tsconfig.json`
- For build or runtime changes:
  - `bun run build:cli`
  - `bun run build:tools` when `packages/tools` changes
- For CLI surface changes:
  - `node packages/cali/dist/index.js --help`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [callstackincubator/cali](https://github.com/callstackincubator/cali) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
