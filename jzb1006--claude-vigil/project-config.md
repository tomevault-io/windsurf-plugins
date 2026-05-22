---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vigil is a CLI that runs `claude -p` unattended against a queue of tasks. Each task gets its own git worktree on a `vigil/<id>-<slug>` branch (slug derived from `task.title` or the prompt; falls back to plain `vigil/<id>` when neither is meaningful). The source repo is never modified directly. When a task finishes the runner auto-commits anything Claude left uncommitted and pushes the branch to `origin` if it's configured. Quota handling is purely reactive: a task only pauses when Anthropic flips a limiter into a pausing status (`"blocked"` — window crossed — or `"rejected"` — this specific request was just refused with a 429; `PAUSE_STATUSES` in `models.py` is the canonical set), and resume is scheduled at the pausing limiter's `resets_at + 60s` buffer.

## Commands

```bash
uv sync --extra dev          # install with test deps
uv run pytest -q             # full test suite
uv run pytest tests/test_queue.py::test_next_runnable_respects_dependencies   # single test
uv run ruff check .          # lint
uv run mypy src/             # type-check (strict)

# CLI (from an editable install)
uv tool install --editable .
vigil doctor                 # pre-flight checks
vigil calibrate              # one-shot probe of `claude -p`
vigil add '<prompt>' --model sonnet --scope-write 'src/auth/**' --budget-cost 1.5
vigil start                  # foreground; SIGINT or `vigil stop` exits cleanly
```

`README.md` and `README.zh.md` are the user-facing docs and they document every flag and design choice in detail — read them before redesigning behavior.

## State layout (split intentionally)

Two roots, owned by `vigil/config.py`:

- **`VIGIL_HOME`** = `platformdirs.user_data_dir("vigil")` (e.g. `~/Library/Application Support/vigil/` on macOS) — internal state: `vigil.db`, `reports/`, `STOP`, `calibration.json`, `config.json`.
- **`VIGIL_USER_HOME`** = `~/.vigil/` — developer-facing artifacts: `worktrees/<id>/` and `logs/`. Short, no-spaces path so `cd` and `tail` are ergonomic on every OS.

Both are overridable via env vars of the same names — required for isolated demos / tests (running `vigil clear` against your real queue is otherwise unrecoverable). The override is module-level, so it must be set on the process that imports `vigil.config`.

Anything you put under platformdirs is hidden; anything under `~/.vigil/` is meant to be opened by the developer. Don't move worktrees or logs back into platformdirs.

## Architecture: how a task flows

1. **`vigil/cli.py`** is the only entry point (Click group). It calls `ensure_dirs()` + `configure_logging()` before any subcommand and resolves the active language via `i18n.resolve_lang` (CLI flag > `VIGIL_LANG` > config file > `$LANG` > default).
2. **`vigil/queue.py`** is the SQLite source of truth. All status transitions (`pending → running → done/failed/needs_review/...`) go through `TaskQueue.update`. `next_runnable()` is DAG-aware (a task is only runnable when every `depends_on` is `DONE`). All datetimes in the DB are naive UTC — `_to_naive_utc()` normalizes anything tz-aware before insert.
3. **`vigil/runner.py`** owns the per-task lifecycle. `main_loop` installs SIGINT/SIGTERM handlers, runs crash recovery (any task left `RUNNING`/`WRAPPING_UP` from a killed daemon is marked `INTERRUPTED`), and alternates between `runner.run(pending)` and `runner.resume(paused)`.
4. **`vigil/worktree.py`** creates `~/.vigil/worktrees/<id>/` on a fresh `vigil/<id>-<slug>` branch from a **locked base SHA**. `derive_branch_name(task.id, task.title or task.prompt)` builds the branch (`slugify` keeps ASCII alnum + CJK + `-_`, collapses runs of `-`, truncates to 40 chars; empty slugs fall back to `vigil/<id>`). The stored `task.branch` is authoritative — the legacy `branch_name(task_id)` helper survives only as a fallback for pre-rename rows. Where the base SHA comes from depends on `vigil add` flags:
   - default (no flags) → `resolve_current_head(cwd)` captures the source repo's current HEAD as a SHA at add time
   - `--base <ref>` → `resolve_base(cwd, ref)` resolves the given branch/tag/SHA at add time
   - `--depends-on <id>` → `task.base_sha` left NULL at add time; the runner resolves the dep's branch tip via `_dep_branch_name(dep_id)` (reads `dep.branch` from the DB, falls back to `vigil/<dep_id>` for legacy rows) right before creating the worktree, then persists it back
   Critical: `resolve_*` always returns a **SHA**, not a symbolic ref — `HEAD` inside a worktree advances as the agent commits, so we have to capture the starting commit at creation time for `diff_stats` to be accurate. `--base` and `--depends-on` are mutually exclusive at the CLI level.
5. **`vigil/claude.py`** spawns `claude -p` with `--print --output-format stream-json --verbose --permission-mode bypassPermissions --disallowed-tools AskUserQuestion --allow-dangerously-skip-permissions`. `StreamSession.events()` yields parsed JSON dicts plus two synthetic events: `_parse_error` and `_exit`.
6. The runner's stream loop (`_spawn_and_stream`) feeds each event through:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jzb1006/claude-vigil](https://github.com/jzb1006/claude-vigil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
