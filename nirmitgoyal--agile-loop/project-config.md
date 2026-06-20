---
trigger: always_on
description: Run the queued GSD → Superpowers → review → ship loop end-to-end. Reads tasks from `docs/agile-loop/tasks/*.md`, spawns isolated child sessions per stage, opens one PR per task, and auto-merges it (squash) before continuing to the next task. Use when asked to "run the agile loop", "process the next queued task", or "drive the autonomous engineering loop".
---


# Agile Loop — Claude Code adapter

You are the Claude Code adapter for Agile Loop. The Codex adapter lives at `scripts/agile-loop.sh` and runs the same contract via `codex exec --ephemeral`; you run it via `claude -p` headless child sessions. Both adapters read the same `docs/agile-loop/tasks/*.md` queue and write the same `.agile-loop/status.json` (the dashboard at `scripts/agile-dashboard.py` works on either path). `references/prompts.md` is shared documentation of the host-neutral prompt shapes; the templates Claude actually sends are inlined in **Prompt templates** below so there is no runtime file lookup against the target repo.

When you spawn a child via `claude -p "<prompt>"`, that is the Claude-side equivalent of `codex exec --ephemeral`: no chat history carryover, child reconstructs all context from the repository, task file, and explicit output files.

## Pre-flight

1. Parse args (all optional): `--repo PATH` (default `$PWD`), `--base BRANCH` (default `main`), `--max-iterations N` (default 10), `--unsafe-bypass-approvals` (boolean), `--dry-run` (boolean), `--dashboard-host HOST` (default `127.0.0.1`, env `AGILE_LOOP_DASHBOARD_HOST`), `--dashboard-port N` (default `8765`, env `AGILE_LOOP_DASHBOARD_PORT`), `--no-dashboard` (boolean, env `AGILE_LOOP_NO_DASHBOARD=1`). `cd` into the resolved repo root and remember it as the absolute `REPO`.
2. Resolve approval bypass: live runs require either `--unsafe-bypass-approvals` or `AGILE_LOOP_UNSAFE_BYPASS=1` in the environment. If neither is set and `--dry-run` is also not set, stop with a clear blocked message — the Codex adapter has the same gate (`scripts/agile-loop.sh`).
3. Confirm `git`, `gh`, `claude`, and `python3` are on `PATH`. If any are missing, write a blocked status with a clear message and stop.
4. Generate a `RUN_ID` (UTC timestamp + short random). Create `.agile-loop/` and `.agile-loop/runs/<RUN_ID>/` if missing. Remember these absolute paths for the whole run: `RUN_DIR=$REPO/.agile-loop/runs/$RUN_ID` and `WORKTREES=$REPO/.claude/worktrees`. **`.agile-loop/`, `.agile-loop/status.json`, and `docs/agile-loop/tasks/*.md` always live in the main repo** (never inside a worktree) so the dashboard keeps reading them and they survive worktree teardown. Each claimed task runs in its own Claude worktree under `WORKTREES` (see **Per-iteration loop → 1. Pick the next task**). `.claude/worktrees/` lives inside the repo, but git treats a registered worktree path as a nested worktree — it does not appear as untracked in the main repo — so no `.gitignore` change is needed.
5. Initialize `.agile-loop/status.json` with `status: starting` via the status writer (see **Status writer** below).
6. **Auto-start the dashboard** so `http://<dashboard-host>:<dashboard-port>` (default `http://127.0.0.1:8765`) always reflects the active loop. See **Dashboard auto-start** below for the exact behavior. Skip this step entirely when `--no-dashboard` / `AGILE_LOOP_NO_DASHBOARD=1` is set.
7. Print a one-line summary to the user: `Agile Loop: repo=<repo> base=<base> max=<n> dry_run=<bool> run=<RUN_ID> dashboard=<url|skipped>`.

## Child-session invocation

For every agent-backed step, spawn one fresh child via `Bash`, with its working directory set to the current task's worktree (`$WORKTREE`, created during claim):

```bash
( cd "$WORKTREE" && claude -p "$(cat "$RUN_DIR/<stage>.prompt.md")" \
  --model <stage-model> \
  --output-format text \
  --dangerously-skip-permissions \
  > "$RUN_DIR/<stage>.output.md" \
  2> "$RUN_DIR/<stage>.events.log" )
```

- The child runs **inside the task's Claude worktree** (`cd "$WORKTREE"`), so all of its file edits and commits land on the task branch and never touch the main checkout. The prompt `cat`, stdout `>`, and stderr `2>` all use **absolute** `$RUN_DIR` paths, so run artifacts stay in the main repo even though cwd is the worktree.
- `--dangerously-skip-permissions` is required so the child can write files and run shell commands without prompting. Only pass it when the parent loop is gated by `--unsafe-bypass-approvals` / `AGILE_LOOP_UNSAFE_BYPASS=1`. In `--dry-run` mode, do not spawn the child at all — log "DRY RUN: would run stage=<stage>".
- Both `--model <stage-model>` and `--effort <stage-effort>` are mandatory on **every** stage — never let a stage silently inherit an ambient default model or effort. Resolve both from the stage's row in **Model and effort routing** below and pass them explicitly on each `claude -p` call.
- Build each prompt file first using the inline templates in **Prompt templates**.

## Model and effort routing

Each stage pins its own model and effort — there is no single default. Most stages run on the latest Opus; the ship and auto-merge stages run on the latest Sonnet. Effort is set per stage (`max`, `xhigh`, or `medium`) and, for deep review, varies by pass.

- **Plan → latest Opus, `--effort max`.** Full power deciding the approach up front.
- **Implementation → latest Opus, `--effort medium`.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nirmitgoyal/agile-loop](https://github.com/nirmitgoyal/agile-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
