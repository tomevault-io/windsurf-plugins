---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Wikifier is a **zero-dependency, agent-to-agent codebase wiki**: it gives AI agents a token-efficient, queryable map of a codebase (health matrix, dependency graph, per-file wiki summaries) so they can look things up instead of re-reading full sources. The human layer (`index.html` dashboard) is strictly secondary — a read-only viewer of the same artifacts. It is deliberately NOT a general human documentation tool.

**This repo dogfoods itself.** Wikifier runs on its own source, and you are expected to follow its agent protocol when working here (see below). The authoritative agent contract is `skills/run.md` (Agent Protocol v0.5) — read it before substantive work.

## Hard Constraints

- **NON-NEGOTIABLE: effectively zero-dependency.** The core must run on pure Python stdlib (>=3.8) + POSIX shell alone — `pyproject.toml` declares `dependencies = []` and that stays. The reason is architectural, not stylistic: anyone forking this project must be free to bring their own third-party libraries on top of a dependency-free base. Therefore: never add a runtime dependency, never couple core logic to an optional extra (the `mcp` extra must remain strictly optional and isolated to `wikifier/mcp/`), and never make core behavior degrade when nothing beyond the stdlib is installed.
- **`library.md`, `file_health.md`, `pending_updates.md` are generated/managed artifacts.** Never hand-edit `library.md` (overwritten by `update-maps`); mutate health/pending only through the official commands (they hold locks).
- **Human-layer separation:** only `index.html` is deployed to target projects on `init`. `diagnostics.html` is maintainer-only and stays in this repo.
- Agent-to-agent scope only — do not grow features toward general human docs or IDE use.

## Mandatory Workflow (the project's own protocol)

After any source edit in this repo:

```bash
wikifier check-changes                                   # what's dirty; read file_health.md + pending_updates.md
# ... edit source (prioritize 🔴 Red, then 🟡 Yellow) ...
wikifier record-change "path/to/file" "why (semantic reason, include subid for agent work)"   # REQUIRED — never skip
# ... update the file's wiki entry (*.wiki.md or library summary) ...
wikifier mark-green "path/to/file" "reason"
wikifier update-maps                                     # only when imports/structure changed
wikifier health --summary                                # re-validate at end of session
```

`record-change` is the semantic audit trail (journal + health + pending) — skipping it breaks the system's core value. For edits to agent-maintained docs (skills/run.md, README, Findings/M5-*.md), precede with "FRESH 3" hygiene: grep the target for 0 stale/deferred matches first.

The Wikifier MCP server may be connected (tools like `get_project_status`, `get_dependencies`, `get_file_wiki`, `record_change`, `mark_green`). Prefer MCP tools when available; the CLI/library is the battle-tested fallback (MCP can time out on very large barrel-heavy targets). For any external project, always pass explicit `project_root=` or `WIKIFIER_PROJECT_ROOT=`.

## Commands

```bash
pip install -e .                       # dev install (entry points: wikifier, wikifier-mcp)
python -m wikifier <command>           # equivalent to the wikifier entry point
./wikifier.sh <command>                # shell launcher (root copy; packaged copy in wikifier/scripts/)
wikifier init [--target DIR]           # bootstrap a target project (copies only index.html)
wikifier update-maps [--full] [--directory=src/] [--max-files=N]   # rebuild dependency graph + library.md (pure-Python pipeline; wikifier.sh delegates here — the in-shell first-pass was retired, --sh is a deprecated no-op)
wikifier health [--summary|--json]     # health matrix (use --summary/--json for machine consumption)
wikifier monitor &                     # background incremental heartbeat
wikifier daemon {start|stop|status}    # managed background maintenance
WIKIFIER_PROJECT_ROOT=/abs/path wikifier-mcp   # MCP server (requires pip install wikifier[mcp])
./scripts/publish.sh [test|prod]       # PyPI release (build + twine)
```

**Tests:** `python -m unittest discover tests` (pure stdlib unittest — no pytest, per the zero-dependency rule; ~49 tests covering parsers, cache schema, cycles, barrel invalidation, health workflow, gap-closure hygiene). Run it after any core change. Additionally verify by dogfooding: `wikifier check-changes`, `wikifier update-maps` (catches parse/pipeline errors), `wikifier health --summary`, and MCP smoke calls (`get_project_status`, `suggest_next_actions`), then confirm the artifacts (file_health.md, library.md) look right.

## Architecture

Pipeline: **scan → parse → resolve → cache → health → artifacts**, all under `wikifier/` (~19.8k LOC pure stdlib; package version in `wikifier/__init__.py`, currently 4.5.x).

- `cli.py` — entry point, unified project-root discovery (env var → `.wikifier/`/marker walk → `.git`/manifest markers → cwd), and `run_full_update()` — the pure-Python pipeline (dirty detection → parsers → persist → cycles → ACS).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IronAdamant/wikifier](https://github.com/IronAdamant/wikifier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
