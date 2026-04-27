---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS

Guidance for coding agents working in this repository.

## Project Shape

- TypeScript / Node.js / pnpm project
- Personal AI assistant focused on long-running, observable task execution
- Feishu/Lark is currently the only supported channel
- The codebase is early-stage and should be changed incrementally

## Core Rules

- Make the best change that solves the task.
- Preserve existing user work.
- Prioritize code quality, maintainability, extensibility, and product quality above short-term shortcuts.
- Do not reject a refactor just because it is speculative; use judgment and make the change when it improves the result materially.
- If behavior changes, add or update tests.
- If user-facing setup or usage changes, update the relevant repository docs in the same change when practical.
- Keep explanations concrete and tied to actual code paths.

## File And Git Safety

- Never use destructive git commands such as `git reset --hard` or `git checkout --` unless explicitly requested and approved.
- Do not amend commits unless the user explicitly asks.
- Never revert changes you did not make.
- If you notice unrelated unexpected modifications, stop and ask how to proceed if they conflict with the task.
- Prefer `apply_patch` for manual file edits.
- Do not use Python for file editing when a shell command or `apply_patch` is sufficient.
- Prefer ASCII when editing files unless the file already uses non-ASCII and there is a clear reason to match it.

## Dependency Commands

- When running `pnpm add`, `pnpm install`, or similar dependency-changing commands:
  - if the command hits a permissions or sandbox problem, request escalated permissions directly
  - do **not** first try to work around it with `--store-dir` or other local store rewrites
  - reason: this repository has repeatedly hit false-detour issues from trying to patch pnpm store behavior instead of just requesting the needed permission

## Commands

- `pnpm install` - install dependencies
- `pnpm build` - TypeScript check (`tsc --noEmit`)
- `pnpm typecheck` - same as `pnpm build`
- `pnpm lint` - Biome checks
- `pnpm format` - Biome write/fix
- `pnpm test` - Vitest suite
- `pnpm test:integration` - integration tests
- `pnpm preflight` - build + format + lint + test
- `pnpm start` - run the app once
- `pnpm dev` - run the app with watch mode
- `pnpm meditation:backtest -- --tick-at <iso> [--lookback-days N | --start-at <iso> --end-at <iso>] [--label name]` - run the normal meditation pipeline against a sandbox copy of the live DB/workspace for replay/backtest without mutating live meditation state
- `pnpm meditation:episode-study -- --tick-at <iso> [--lookback-days N | --start-at <iso> --end-at <iso>] [--session-id <id>]... [--top-sessions N] [--label name]` - compare multiple candidate friction-episode extraction strategies against real historical session data and write reports under `.tmp/meditation-episode-study/`

Prefer the narrowest relevant test command first. Run `pnpm preflight` for larger or user-facing changes.

## Meditation Backtest

- The manual replay entrypoint is [src/script/meditation-backtest.ts](src/script/meditation-backtest.ts).
- It reuses the same `MeditationPipelineRunner` as production cron runs.
- It prepares an isolated sandbox under `.tmp/meditation-backtests/<label>/` with:
  - a copied SQLite database
  - copied shared/private memory files
  - isolated meditation logs and daily notes
- The backtest runner also hard-denies the live `~/.pokoclaw` data tree at the meditation security layer so the replay cannot read or write live runtime data during the run.
- Use this when iterating on meditation prompts, clustering, evaluation, or rewrite behavior so you do not need to wait for the live cron schedule or clean live state between runs.
- `pnpm meditation:backtest -- --help` prints the full CLI usage from the script itself.
- Prefer explicit `--start-at/--end-at` when you want deterministic replay of one known window.
- Prefer `--lookback-days` when you want a quick sandboxed approximation of the normal production lookback behavior.
- Common usage:
  - `pnpm meditation:backtest -- --tick-at 2026-04-15T12:30:00.000Z --lookback-days 7`
  - `pnpm meditation:backtest -- --tick-at 2026-04-15T12:30:00.000Z --start-at 2026-04-14T00:00:00.000Z --end-at 2026-04-15T00:00:00.000Z --label github-task-window`
  - `pnpm meditation:backtest -- --tick-at 2026-04-15T12:30:00.000Z --lookback-days 7 --last-success-at null`
- Important flags:
  - `--tick-at`: logical run time for the replay; defaults to now when omitted
  - `--start-at` / `--end-at`: explicit UTC window bounds for deterministic replay
  - `--lookback-days`: fallback lookback horizon when `--start-at` is omitted
  - `--last-success-at`: override production `last_success_at` semantics without touching live state; pass `null` to force lookback mode
  - `--label`: names the sandbox folder under `.tmp/meditation-backtests/`
- Typical inspection flow:
  - run the replay with a stable `--label`
  - inspect `.tmp/meditation-backtests/<label>/logs/meditation/...`
  - inspect `.tmp/meditation-backtests/<label>/workspace/meditation/<date>.md`
  - delete the sandbox folder when the replay is no longer needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielwpz/pokoclaw](https://github.com/danielwpz/pokoclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
