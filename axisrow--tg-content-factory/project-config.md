---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Run the web server — spawns the embedded Telegram worker by default so a
# single command gives you UI + actual collection (#457 round 4). For split
# deployments (Docker/k8s) pass --no-worker and run `worker` separately.
python -m src.main serve [--web-pass PASS] [--no-worker]

# Standalone Telegram worker — only needed alongside `serve --no-worker`.
python -m src.main worker

# Lint
ruff check src/ tests/ conftest.py

# Auto-fix lint issues
ruff check --fix src/ tests/ conftest.py

# Run parallel-safe tests (-n auto is capped at 4 workers by root conftest.py;
# override via TGCF_PYTEST_XDIST_WORKERS; single-test runs with :: force 1 worker)
pytest tests/ -v -m "not aiosqlite_serial" -n auto

# Run aiosqlite-backed tests serially
pytest tests/ -v -m aiosqlite_serial

# Run a single test
pytest tests/test_web.py::test_health_endpoint -v

# Benchmark serial vs safe mixed-mode suite execution
python -m src.main test benchmark
```

Full CLI reference:

```bash
python -m src.main [--config CONFIG] serve [--web-pass PASS]
python -m src.main [--config CONFIG] worker
python -m src.main [--config CONFIG] collect [--channel-id ID] | collect sample
python -m src.main [--config CONFIG] search "query" [--limit N] [--mode MODE]

python -m src.main messages read <identifier> [--limit N] [--live] [--phone PHONE] [--query TEXT] [--date-from DATE] [--date-to DATE] [--topic-id ID] [--offset-id ID] [--format text|json|csv]
python -m src.main channel list|add|delete|toggle|collect|stats|refresh-types|refresh-meta|import|add-bulk|list-for-import|tag (tag list|add|delete|set|get)
python -m src.main filter analyze|apply|reset|precheck|toggle|purge|purge-messages|hard-delete
python -m src.main account list|info|toggle|set-primary|delete|send-code|verify-code|add|flood-status|flood-clear
python -m src.main scheduler start|trigger|status|stop|job-toggle|set-interval|task-cancel|clear-pending|queue-pause|queue-resume
python -m src.main notification setup|status|delete|test|dry-run|set-account
python -m src.main test all|read|write|telegram|benchmark

python -m src.main stop|restart
python -m src.main search-query list|get|add|edit|delete|toggle|run|stats
python -m src.main pipeline list|show|add|dry-run-count|edit|delete|toggle|run|generate|generate-stream|runs|run-show|queue|moderation-list|moderation-view|publish|approve|reject|bulk-approve|bulk-reject|refinement-steps|export|import|templates|from-template|ai-edit|filter|node|edge|graph
python -m src.main photo-loader dialogs|refresh|send|schedule-send|batch-create|batch-list|items|batch-cancel|auto-create|auto-list|auto-update|auto-toggle|auto-delete|run-due
python -m src.main dialogs list|refresh|resolve|leave|join|topics|cache-clear|cache-status|send|forward|edit-message|delete-message|create-channel|create-group|pin-message|react|unpin-message|download-media|participants|edit-admin|edit-permissions|kick|broadcast-stats|archive|unarchive|mark-read|queue|status|cancel|clear-pending
python -m src.main agent threads|thread-create|thread-delete|chat|thread-rename|thread-stop|messages|context|test-escaping|test-tools
python -m src.main analytics top|content-types|hourly|summary|daily|pipeline-stats|trending-topics|trending-channels|velocity|peak-hours|calendar|trending-emojis|channel
python -m src.main provider list|add|delete|probe|refresh|test-all
python -m src.main export json|csv|rss
python -m src.main translate stats|detect|run|message
python -m src.main settings get|set|info|server-time|agent|filter-criteria|reactions|semantic
python -m src.main debug logs|memory|timing
python -m src.main image generate|models|providers|generated
python -m src.main mcp-server [--no-pool]   # expose agent tool registry as stdio MCP server (for external agents like Codex)
```

CLI command for Telegram dialogs management is `dialogs`.

## Architecture

Three layers: **CLI/Web** → **Telegram + Search + Scheduler + Agent/Pipeline** → **SQLite**

- **Runtime split (web ↔ worker)**: since #444 the runtime consists of two `AppContainer` flavours keyed on `runtime_mode` ("web" vs "worker") in `src/web/bootstrap.py`. Since #457 round 4 they normally run in the **same process**: `serve` spawns an `EmbeddedWorker` (`src/web/embedded_worker.py`) as an asyncio task next to the web container. Pass `--no-worker` to run only the web side and start `python -m src.main worker` separately (Docker/k8s split deployments).
  - Web container (`runtime_mode="web"`) uses snapshot shims (`SnapshotClientPool`, `SnapshotCollector`, `SnapshotSchedulerManager` in `src/web/runtime_shims.py`). It does NOT open Telegram connections; UI actions enqueue work into `collection_tasks` / `telegram_commands` / task tables and read `runtime_snapshots` to render status.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axisrow/tg_content_factory](https://github.com/axisrow/tg_content_factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
