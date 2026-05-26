---
trigger: always_on
description: Instructions for AI coding assistants working on this codebase.
---

# Kouhai Bot — Development Guide

Instructions for AI coding assistants working on this codebase.

## Architecture

```
NapCat (QQ) ──WS──> worker.py
                         │
                         ├── process_event(..., spawn_handlers=True)
                         ├── cmd/*.py  auto-discovered by registry
                         └── scheduler/ background loop (60s tick)
```

## Key Design Decisions

- **Command auto-discovery**: Each command in `handlers/cmd/*.py` calls `register()` at module load. The `registry.discover_commands()` scans with `pkgutil.iter_modules`. Adding a new command = create a `.py` file with a `register()` function.
- **Limited aliases**: Only five short aliases are supported: `/newproblem`→`/np`, `/problem`→`/pb`, `/submit`→`/sbm`, `/review`→`/rv`, `/clarify`→`/clrf`. Old aliases such as `/sb`, `/排名`, and Chinese aliases remain unsupported. New commands default to `aliases=[]` unless explicitly approved.
- **Help auto-generation**: `handlers/cmd/help.py` reads `registry.all_commands()` and builds the help text dynamically. Descriptions must match old bridge.py wording.
  `usage` field = args suffix in /help display (e.g. `usage="你的做法"` → `/submit 你的做法`).
- **Scheduler current-group config**: `~/.kouhai-bot/scheduler_config.json` stores job list + time overrides for `CURRENT_GROUP`. Jobs are defined in `scheduler/jobs.py`.
- **Command event log**: `eventlog.py` writes append-only JSONL command events by real local date. `achievements.py` reads those events for the 04:00-to-04:00 daily report. `eventlog_backfill.py` and `tools/backfill_command_events.py` can reconstruct recent saved submit/clarify/review events from `scoreboard.json`.
- **Formula VL**: `problems/fetcher.py` handles CF formula images → Qwen-VL → inline LaTeX. Has white-bg preprocessing, hallucination detection, retry.
- **Stale cache detection**: `picker.py:fetch_statement()` detects caches created before VL pipeline via `_vl_processed` flag. Stale caches with images are re-fetched with Qwen-VL. Problems with non-formula images (tex-graphics / diagrams) are skipped.
- **No hermes cron involvement**: The bot runs its own scheduler loop (`scheduler/engine.py`), not hermes cron jobs.
- **Single worker runtime**: `worker.py` keeps the NapCat reverse-WS connection, dispatches commands, and owns the scheduler in one process. There is no SQLite event queue, ingress supervisor, worker hot-swap, or auto-update loop.
- **User groups**: `user_groups.py` — all users default to `default`; `USER_GROUPS` configures
  non-default groups such as `starred`/`打星`, their members, submit delay, and rejection
  message. `do_daily_post` writes `state.json` `posted_at`; if missing, cooldown falls back
  to matching `daily_msg.json` mtime.
- **Curfew (宵禁)**: `curfew.py` — `/submit` is blocked during a daily quiet window defined
  by `CURFEW_START_HOUR` and `CURFEW_DURATION_HOURS`. Other commands (clarify, review,
  scoreboard, etc.) are unaffected. Curfew wraps past midnight correctly (e.g. start=22,
  duration=6 → 22:00–04:00).
- **LLM fallback**: `llm.py` — providers are tried in list order defined by
  `llm.providers` in `config.yaml`. Each provider is retried internally
  (`llm.max_retries`) before moving to the next. All providers use the
  OpenAI-compatible `/chat/completions` format. Per-task model overrides
  (`judge_model`, `clarify_model`, etc.) are defined per provider. `thinking`
  and `reasoning_effort` are sent unconditionally; unsupported fields are
  silently ignored by upstream APIs.
- **Official CF tutorials**: Scraped editorials live under `{data_dir}/tutorials/{pid}.json`
  (see `tools/scrape_cf_tutorial.py`). Runtime extraction is in `tutorials.py`. On the
  On **new problem** (`do_daily_post` / `/newproblem`), `schedule_prefetch_editorial(pid)`
  starts background translation (using `summary_model`) into `tutorial_translations/` so first AC
  can deliver without waiting. On **first AC**, congrats is sent in `_finalize_submit`, then
  `schedule_post_solve_editorial_followup()` only **delivers** (awaits in-flight prefetch if
  needed). Neither path uses the state scheduler. `/review` uses English editorial in LLM
  context only.

## Configuration

### config.yaml

Runtime config comes from `config.yaml` at the repo root (or set `KOUHAI_CONFIG=/path/to/config.yaml`).
The file is **never committed** — `config.yaml` is in `.gitignore`.
Copy `config.example.yaml` to `config.yaml` and fill in your values.

All providers use the OpenAI-compatible `/chat/completions` endpoint.
``base_url`` should include any version prefix (e.g. `https://api.openai.com/v1`,
`https://api.deepseek.com`); `/chat/completions` is appended automatically.

#### Top-level keys

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `bot_qq` | int | — | Bot's QQ number |
| `napcat_ws_host` | str | `0.0.0.0` | WS listen host — MUST be `0.0.0.0` when NapCat runs in Docker |
| `napcat_ws_port` | int | 8095 | WS listen port |
| `napcat_http_host` | str | `127.0.0.1` | NapCat HTTP API host |
| `napcat_http_port` | int | 3000 | NapCat HTTP API port |
| `current_group` | int | — | QQ group served by the bot (**required**) |
| `data_dir` | str | `~/.kouhai-bot` | Shared data directory |

#### `llm` section


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nerovix/kouhai-bot](https://github.com/Nerovix/kouhai-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
