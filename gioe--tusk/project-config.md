---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

tusker is a portable task management system for Claude Code projects. It provides a local SQLite database, a bash CLI (`bin/tusk`), Python utility scripts, and Claude Code skills to track, prioritize, and work through tasks autonomously.

When proposing, evaluating, or reviewing features, consult `docs/PILLARS.md` for design tradeoffs. The pillars define what tusk values and provide a shared vocabulary for resolving competing approaches.

## Commands

```bash
# Init / info
bin/tusk init [--force]
bin/tusk path
bin/tusk config [key]
bin/tusk setup          # config + backlog + conventions in one JSON call
bin/tusk validate

# Task lifecycle
bin/tusk task-get <task_id>        # accepts integer ID or TASK-NNN prefix form
bin/tusk task-list [--status <s>] [--domain <d>] [--assignee <a>] [--workflow <w>] [--format text|json] [--all]  # list tasks (not the built-in TaskList tool)
bin/tusk task-select [--max-complexity XS|S|M|L|XL]
bin/tusk task-insert "<summary>" "<description>" [--priority P] [--domain D] [--task-type T] [--assignee A] [--complexity C] [--workflow W] [--criteria "..." ...] [--typed-criteria '{"text":"...","type":"...","spec":"..."}' ...] [--deferred] [--expires-in DAYS]
bin/tusk task-start <task_id> [--force]
bin/tusk task-done <task_id> --reason completed|expired|wont_do|duplicate [--force]
bin/tusk task-update <task_id> [--priority P] [--domain D] [--task-type T] [--assignee A] [--complexity C] [--workflow W] [--summary S] [--description D]
bin/tusk task-reopen <task_id> --force

# Dev workflow
bin/tusk branch <task_id> <slug>
bin/tusk commit <task_id> "<message>" "<file1>" ["<file2>" ...] [--criteria <id>] ... [--skip-verify]
# Note: tusk commit prepends [TASK-N] to <message> automatically — do not include it yourself
# Note: always quote file paths — zsh expands unquoted [brackets] as glob patterns before tusk receives them
bin/tusk merge <task_id> [--session <session_id>] [--pr --pr-number <N>]
bin/tusk progress <task_id> [--next-steps "..."]

# Criteria
bin/tusk criteria add <task_id> "criterion" [--source original|subsumption|pr_review] [--type manual|code|test|file] [--spec "..."]
bin/tusk criteria list <task_id>
bin/tusk criteria done <criterion_id> [--skip-verify]
bin/tusk criteria skip <criterion_id> --reason <reason>
bin/tusk criteria reset <criterion_id>

# Dependencies
bin/tusk deps add <task_id> <depends_on_id> [--type blocks|contingent]
bin/tusk deps remove <task_id> <depends_on_id>
bin/tusk deps list <task_id>
bin/tusk deps ready

# Utilities
bin/tusk wsjf
bin/tusk lint
bin/tusk autoclose
bin/tusk backlog-scan [--duplicates] [--unassigned] [--unsized] [--expired]   # → {duplicates:[...], unassigned:[...], unsized:[...], expired:[...]}
bin/tusk test-detect               # → {"command": "<cmd>", "confidence": "high|medium|low|none"}
bin/tusk scaffold-reviewer-prompts # → {created:[...], skipped:[...], skill_dir_missing:bool}
bin/tusk add-lib [--lib <name>] [--repo <owner/repo>] [--ref <branch|tag|sha>]  # → {"lib": "<name>", "tasks": [...], "error": null}
bin/tusk init-fetch-bootstrap      # → {"libs": [{name, repo, tasks, error}, ...]}
bin/tusk init-write-config [--domains <json>] [--agents <json>] [--task-types <json>] [--test-command <cmd>] [--project-type <type>] [--project-libs <json>]  # → {"success": bool, "config_path": "...", "backed_up": bool}
bin/tusk git-default-branch        # → prints default branch name (e.g. "main"); symbolic-ref → gh fallback → "main"
bin/tusk branch-parse [--branch <name>]  # → {"task_id": N}; parses task ID from current or named branch
bin/tusk sql-quote "O'Reilly's book"   # → 'O''Reilly''s book'
bin/tusk shell

# Versioning
bin/tusk version
bin/tusk version-bump                              # increment VERSION by 1, stage, echo new version
bin/tusk changelog-add <version> [<task_id>...]   # prepend dated entry to CHANGELOG.md, echo block
bin/tusk migrate
bin/tusk regen-triggers
bin/tusk upgrade [--no-commit] [--force]  # --no-commit: skip auto-commit; --force: upgrade even if version matches or exceeds remote
```

Additional subcommands (`blockers`, `review`, `chain`, `loop`, `deps blocked/all`, `session-stats`, `session-close`, `session-recalc`, `skill-run`, `call-breakdown`, `token-audit`, `pricing-update`, `sync-skills`, `dashboard`) follow the same `bin/tusk <cmd> --help` pattern — see source or run `--help` for flags.

There is no build step or external linter in this repository.

## Running the test suite

```bash
python3 -m pytest tests/ -v          # run all tests
python3 -m pytest tests/unit/ -v     # unit tests only (pure in-memory, no subprocess)
python3 -m pytest tests/integration/ -v  # integration tests only (requires a working tusk installation)
```

Integration tests initialize their own temporary database automatically via a pytest fixture — no manual `tusk init` is needed.

Dev dependencies (pytest) are listed in `requirements-dev.txt`. Install with:

```bash
pip install -r requirements-dev.txt
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gioe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
