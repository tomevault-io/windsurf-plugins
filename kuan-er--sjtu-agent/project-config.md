---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project summary

SJTU Agent is a deployable campus assistant for Shanghai Jiao Tong University. It provides an LLM-powered CLI chat interface, multi-platform bots (Telegram, Feishu/Lark, WeChat), DDL aggregation across four campus platforms, daily reports, reminder daemons, a news aggregator, a homework agent (Canvas + Claude Code), a web config UI, and an MCP server for AI agent integration.

## Setup & development

```bash
# Initial setup (creates .venv, installs in editable mode, optional Playwright)
bash install/install.sh          # macOS/Linux
powershell -ExecutionPolicy Bypass -File .\install\install.ps1  # Windows

# Manual install
python -m venv .venv && source .venv/bin/activate && pip install -e .

# Configuration
cp .env.example .env     # edit with real credentials
sjtu-agent setup         # interactive config wizard, writes config.json + agent_config.json

# Running
sjtu-agent               # interactive CLI chat (default)
sjtu-agent doctor        # print runtime paths and config status
sjtu-agent web           # open web config UI at localhost:7860
sjtu-agent telegram-bot  # start Telegram bot daemon
sjtu-agent feishu-bot    # start Feishu bot daemon
sjtu-agent wechat-bot    # start WeChat bot daemon
sjtu-agent ddl           # run DDL check once
sjtu-agent daily-report  # generate and send daily report
sjtu-agent remind-check  # run reminder daemon once
sjtu-agent news-digest   # run news digest
sjtu-agent mcp           # start MCP server for DDL queries
sjtu-agent login         # refresh platform cookies via Playwright
sjtu-agent install-daemons  # install background services
sjtu-agent update        # git pull + reinstall
```

## Testing

```bash
pytest                          # run all tests
pytest tests/test_config.py     # run a single test file
pytest -k "test_function_name"  # run tests matching a pattern
```

Tests live in `tests/`, discovered via `pytest.ini`. Configuration is minimal — no coverage or CI workflow is configured yet.

## Environment variables

| Variable | Purpose |
|---|---|
| `JACCOUNT_USERNAME` | jAccount login username for my.sjtu.edu.cn |
| `JACCOUNT_PASSWORD` | jAccount password |
| `ZHIYUAN_API_KEY` | SJTU Zhiyuan No.1 API key (OpenAI-compatible) |
| `ANTHROPIC_API_KEY` | Optional, used for captcha recognition via Claude Haiku |
| `SJTU_AGENT_HOME` | Override the runtime data directory (default: platform-specific user data dir) |
| `SJTU_HOMEWORK_DIR` | Override the homework/assignments directory |

## Architecture

### Package structure (`sjtu_agent/`)

```
sjtu_agent/
  cli.py              # argparse entry point — all subcommands dispatch from here
  paths.py            # centralized runtime paths (DATA_DIR, CONFIG_PATH, etc.) + atomic_write_json
  config.py           # ConfigStore singleton — typed, cached, hot-reloading access to config.json
  setup_wizard.py     # interactive first-run configuration (large, ~54KB)
  terminal_ui.py      # Rich-powered terminal helpers
  homework_agent.py   # Canvas homework fetching + Claude Code analysis

  agent/              # LLM chat engine (refactored from root-level agent.py in 2026-05)
    __init__.py       # re-exports all public symbols
    prompts.py        # SYSTEM_PROMPT, date context builder, tool labels (~22KB)
    tools.py          # TOOLS definitions + all tool_xxx implementations (~172KB — largest file)
    runner.py         # LLM client creation, single-turn execution, streaming
    chat_loop.py      # config loading, chat main loop, startup logic

  scheduler/          # cross-platform daemon manager
    __init__.py       # facade — dispatches to platform backend by sys.platform
    launchd.py        # macOS launchd plist generator
    systemd.py        # Linux systemd unit generator
    taskschd.py       # Windows Task Scheduler integration
    psmuxd.py         # Windows psmux (detached session) integration

  news_aggregator/    # intelligent news collection + ranking system
    aggregator.py     # concurrent multi-source fetching
    profile.py        # user profile learned from chat history
    ranker.py         # LLM-based + keyword news ranking
    digest.py         # markdown + HTML digest builder
    storage.py        # news storage and deduplication
    sources/          # per-platform scrapers (jwc, shuiyuan, official, canvas)

  web/                # local web config UI
    server.py         # pure-Python HTTP server (no framework, ~43KB)
    static/index.html # SPA frontend (~60KB)
```

### Key design patterns

**ConfigStore singleton** (`config.py`): All config access goes through `ConfigStore.get_instance()`, not raw `json.loads()`. It caches values, tracks file mtime for hot reload, and provides typed accessors. Test it by mocking the singleton.

**Atomic writes** (`paths.py`): Always use `atomic_write_json(path, data)` for persistent state files — it writes to a temp file and `os.replace()`s, so crashes never leave half-written files that could trigger "re-send all reminders" bugs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kuan-er/sjtu-agent](https://github.com/kuan-er/sjtu-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
