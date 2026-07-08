---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

`cc-hooks-metrics` gives Claude Code users a **fast, actionable overview of hook health** — so you can immediately see what's broken, slow, or regressing without wading through raw data. Designed to be shareable, not just personal.

## Working in this repo

Discoveries outside the current task go to `TODO.md`, not into the code. Add a one-line description under the appropriate section (Backlog for ideas, Prioritized for approved work). Do not implement without approval.

## Running the report

```bash
# Default: traffic lights, overhead waterfall, action items, trends, guardrails
~/.claude/hooks/hooks-report.sh

# Verbose: adds perf table, WoW summary, top projects, FIXED/GONE trends, + 7 legacy sections
~/.claude/hooks/hooks-report.sh --verbose

# OTel-aligned JSON export
~/.claude/hooks/hooks-report.sh --export

# Show recent sessions
~/.claude/hooks/hooks-report.sh --sessions

# Drill into a specific step
~/.claude/hooks/hooks-report.sh --step audit-logger

# Pipe to Claude for analysis
~/.claude/hooks/hooks-report.sh --export | claude -p "Analyze and suggest next steps"
```

Output is always Rich static. `--export` outputs JSON. `--verbose` adds legacy detail sections.

## Deployment

Scripts are installed to `~/.claude/hooks/` — the copies in this repo are the source of truth. See `TODO.md` for planned distribution improvements (Homebrew, install script).

```bash
# Deploy Python package + wrapper
rsync -a --delete hooks_report/ ~/.claude/hooks/hooks_report/
install -m 755 hooks-report.sh ~/.claude/hooks/hooks-report.sh
```

The database path defaults to `~/.claude/hooks.db` and can be overridden with `CLAUDE_HOOKS_DB`.

## Architecture

**Data flow**: Claude Code event → `hook-metrics.sh` (wrapper) → downstream hook script → `hooks.db`

### Data ingestion scripts (unchanged bash)

`hook-metrics.sh` is a **passthrough wrapper** — takes `EVENT:STEP_NAME` as `$1` and the actual hook script + args as remaining args. Captures wall-clock timing via `/usr/bin/time -p`, git context, and exit code, then inserts a row into `hook_metrics`. The wrapped script's exit code is always preserved.

`audit-logger.sh` reads the Claude tool-use JSON payload from stdin, extracts `tool_name`, `tool_input`, and `session_id`, and inserts into `audit_events`. Echoes stdin through so it can be chained.

`db-init.sh` is sourced by all scripts. Owns the schema and provides:
- `_init_hooks_db` — creates tables or runs `ALTER TABLE` migration (idempotent)
- `_db_exec sql` — write-only helper with `PRAGMA busy_timeout=1000`, stdout suppressed
- `_q sql` — **report-only** read helper using `sqlite3 -separator '|'` with **no** busy_timeout (adding it would emit a result row corrupting pipe reads)
- `_sql_escape` — single-quote doubling via `sed`
- `_maybe_prune_hooks_db` — 1% probabilistic pruning of rows older than 30 days

### Python package: hooks_report/

Rewrite of the original 1331-line bash report in Python (Rich 14.3.3).

```
hooks_report/
  __init__.py       # empty
  __main__.py       # entry: export/static dispatch
  cli.py            # argparse: --export, --export-spans, --verbose, --static, --db, --include-sensitive
  config.py         # STEP_TIMEOUTS, SEMANTIC_EXIT_STEPS, thresholds, SKIP_HOOKS_PATTERN, OTLP constants
  db.py             # HooksDB: typed dataclasses + SQLite queries
  otlp.py           # OTLP/HTTP JSON export: build_otlp_payload(), send_spans(); zero external deps
  render.py         # Rich helpers: fmt_dur, bar_chart, trend_badge, pct_change, traffic_light_grid
  spans.py          # OTel span model: Span dataclass, hook_metric_to_span, audit_event_to_span, spans_to_dict
  static.py         # Rich Console output: compact sections + verbose sections + export_json
  advisor.py        # Tuning suggestions, hot sequences, periodic summaries
```

**hooks-report.sh** is a Python wrapper with venv detection:
```bash
#!/usr/bin/env bash
DIR="$(cd "$(dirname "$0")" && pwd)"
PYTHON="${DIR}/.venv/bin/python3"
[ -x "$PYTHON" ] || PYTHON=python3
PYTHONPATH="$DIR" exec "$PYTHON" -m hooks_report "$@"
```

### Output structure

**Default mode** (~35-50 lines):
1. Traffic-light grid (Reliability / Performance / Broken Hooks / Regressions / Review Gate)
2. Overhead waterfall — top 5 steps by total overhead with proportional bars
3. Action items grouped by step — one entry per step with all issues deduplicated (or "All clear")
4. `section_wow_compact()` — REGR/SLOW trend lines only (FIXED/GONE suppressed in default)
5. Guardrails table with top block reason per guardrail

**--verbose mode** adds compact sections + 7 legacy detail sections:
- `section_perf_compact()` — per-step performance table (last 7d, avg ≥500ms or has timeout, max 12 rows)
- `section_wow_compact()` — full WoW summary + REGR/FIXED/SLOW/GONE trend lines
- `section_projects_compact()` — top 5 repos by overhead
- 7 legacy detail sections

**--export mode** — OTel-aligned JSON, schema `claude.hooks.trends/v1`, metric names `claude.hooks.*`, attributes `hook.step` / `vcs.repository`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starfysh-tech/cc-hooks-metrics](https://github.com/starfysh-tech/cc-hooks-metrics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
