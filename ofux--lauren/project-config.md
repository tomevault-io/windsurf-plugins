---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
npm run build       # tsc → dist/, then chmod +x the lauren bin
npm run watch       # tsc --watch
npm run clean       # remove dist/
npm run check       # biome check --write (lint + format + organize-imports)
npm run lint        # biome lint only
npm run format      # biome format --write
npm test            # vitest run (one-shot)
npm run test:watch  # vitest in watch mode
```

Tests live next to source as `*.test.ts` (Vitest, ESM, Node `>=20`). Verify changes by `npm run build && npm run check && npm test`. Manual smoke test: from a throwaway repo, run `lauren vibe --dry-run` to print the queue, and `lauren --list` to inspect state (plain table; omit `--list` for the interactive TUI).

## External binaries

The runtime shells out to two CLIs that must be on `$PATH`:
- `claude` — used interactively (planning, spec) and as the default agent for the implement / fix pipeline phases, brain JSON decisions, and merger conflict resolution.
- `codex` — used as the default agent for the review pipeline phase (`codex exec review -o <file> <prompt>`).

Per-phase agent selection is configurable in `.lauren/config.json` under
`agents`: each of `implement`, `review`, `fix`, `merger`, and `brain`
independently accepts `"claude"` or `"codex"`. The adapters live in
`src/agents/` and implement the `CodingAgent` port (`runEdit`, `runReview`,
`runJson`) — see `src/agents/types.ts`.

If a binary is missing for any role configured to use it, `lauren vibe` will
fail at the corresponding step.

## Architecture

Lauren is a single-daemon system that drains a plan queue end-to-end without further human input.

**`lauren` CLI (`src/bin/lauren.ts`)** — the single public executable. `lauren plan` spawns interactive `claude` with `PLAN_SYSTEM_PROMPT`; that claude session writes `.lauren/plans/<slug>.md` and re-invokes the CLI as `lauren _register <slug> --path ... --title ...` (a hidden subcommand) to add a row with status `enqueued` to `.lauren/plans.json`. The default `lauren` (no subcommand) is an interactive Ink TUI (`src/tui/TodoApp.tsx`) showing the queue; `--list` (or non-TTY) prints a static colored table instead. Selecting a row dispatches to `cancelPlan` (`src/cancel.ts`) which routes by status (see *Cancellation* below). The TUI also supports two queue-level actions: `t` on a `failed` row resets it to `ready` via `retryPlan` (`src/retry.ts`), and `r` triggers a brain reorganize pass that re-thinks the whole `ready` queue (refused while `lauren vibe` is running, detected via `.lauren/vibe.pid`).

**Agent surface (`src/init-config.ts` + `src/init-claude.ts` + `src/init-codex.ts` + asset modules)** — `lauren init` first runs an interactive project config setup, offering only installed supported agent CLIs from `$PATH` and writing `.lauren/config.json`; it then installs assets for both Claude Code and Codex. Subcommands `lauren init claude` / `lauren init codex` target only one asset surface and do not run config setup. For Claude Code (`src/claude-assets.ts`), it writes two files into the target's (or user's, with `--global`) `.claude/`: a `lauren` skill that auto-activates on intents like *"add this to lauren"*, and a `/lauren` slash command for explicit invocation. For Codex (`src/codex-assets.ts`), it writes one file into `.agents/skills/lauren/SKILL.md` (or `~/.agents/...` with `--global`) — Codex CLI doesn't support custom slash commands, so the skill is the only surface. All three assets inline the same `PLAN_SYSTEM_PROMPT` — single source of truth in `src/lauren-prompts.ts`. The repo commits canonical copies at `.claude/commands/lauren.md`, `.claude/skills/lauren/SKILL.md`, and `.agents/skills/lauren/SKILL.md`; drift tests in `src/init-claude.test.ts` and `src/init-codex.test.ts` enforce byte-equality with the TS constants. Shared collision/write logic lives in `src/init-common.ts`.

**`lauren vibe` (`src/vibe-command.ts` + `src/watcher.ts`)** — the unified daemon. Each loop iteration (a) drains every `enqueued` plan via *brain* (`brainPlacePlan` in `src/brain.ts`, called from `processEnqueuedPlan` in `src/organize.ts`), transitioning each row from `enqueued` → `preparing` → `ready` in place; then (b) claims one `ready` plan (status → `implementing`), runs the 4-step pipeline in `src/executor.ts`, and marks `done` (or `failed`). Renders progress via Ink (`src/tui/App.tsx` + `runtime.ts`) with distinct UI for the `organizing` and `implementing` phases. On Ctrl-C it demotes the in-flight plan back to `ready` so it can resume cleanly.

### The 4-phase pipeline (`src/executor.ts`)

For each work unit (a whole plan, or a single Step section within a plan):

1. **implement** — `claude -p` with `implementPrompt`/`implementPlanPrompt`. If claude exits 0 but produces no diff, the unit short-circuits as "already done": review/fix/commit are marked `skipped`, the Step row finalizes as `done` with `commit_subject: null`, and the executor moves on. This lets the queue drain past Steps that a human or another agent already implemented instead of getting stuck.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ofux/lauren](https://github.com/ofux/lauren) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
