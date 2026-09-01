---
trigger: always_on
description: This document defines mandatory repository rules for Claude Code contributors in this project.
---

# CLAUDE.md

This document defines mandatory repository rules for Claude Code contributors in this project.
Treat this file as a team contract, not a suggestion list.

## 1) Mission and Scope

Workmate is an LLM-driven agent platform built around FastAPI + LangChain/LangGraph + MCP.
It supports:

- DeepAgent streaming workflows
- Desktop client runtime (`desktop/`)
- WeChat / WeChat Work automation
- Scheduler automation with execution tracking
- Multi-skill prompt/tool orchestration
- MCP-based filesystem and external tool integration

Primary backend entry is `serve.py` (default port `8009`).

## 2) Non-Negotiable Team Rules

1. Never leak or commit secrets (`.env`, API keys, credentials, DB passwords, private tokens).
2. Never hardcode machine-specific absolute paths unless the feature explicitly requires local runtime paths.
3. Keep behavior backward compatible unless explicitly asked to break it.
4. Minimize change surface; avoid unrelated refactors in feature/fix PRs.
5. Prefer configuration-driven behavior over hardcoded branch logic.
6. Keep comments and docs in English unless a file is intentionally Chinese-facing.
7. If uncertain about behavior, read code first and verify with executable checks.

## 3) Repository Map (Current)

### Core Runtime

- `serve.py`: FastAPI application assembly entry (startup init, middleware, router mounting, scheduler/admin mounting)
- `api/routers/`: route modules for desktop, quotation(SSE), mcp, skills
- `api/lifecycle.py`: FastAPI lifespan hooks (startup preload, MCP runtime self-check, shutdown cleanup)
- `api/runtime_config.py`: runtime env/config apply helpers (cloud config injection, output dir/wechat runtime sync)
- `api/runtime_state.py`: process runtime state (preload state, stream stop flags, chat locks/paths)
- `main.py`: CLI execution and scheduled invocation entry
- `workflow/`: DeepAgent execution, model binding, MCP middleware/client, infrastructure, config

### Desktop Stack

- `desktop/webview_shell.py`: PyQt desktop shell
- `desktop/ui/`: local SPA assets (`app.js`, `js/`, `css/`)
- `desktop/README.md`: desktop run/package operational details

### Admin + Scheduler

- `admin_api/`: admin/auth/config/skills server-side APIs
- `admin_web_v/apps/web-ele/`: admin frontend
- `scheduler/`: scheduler APIs, parser, executor, persistence, notification

### Integrations

- `wechat_work/`: WeChat Work WebSocket client and message handling
- `mcp_filesystem/`: local MCP server (document, file, Windows automation, media)
- `skills/`: skill packs
- `prompt/`: prompt templates

## 4) API Surface (Must Stay Consistent)

At minimum, verify compatibility for these route groups when editing app assembly/router flow (`serve.py`, `api/routers/*`, lifecycle hooks):

- `POST /quotation`
- `GET /agent/health`
- Desktop APIs under `/api/desktop/*`
- MCP config APIs under `/mcp/*`
- Skills APIs under `/skills/*`
- Scheduler APIs under `/scheduler/*`
- Admin router mounted via `admin_api`

If route behavior changes, update docs/tests in the same change.

## 5) Configuration and Environment

Single source of truth:

- Root `.env`
- `workflow/config.py`
- `config/mcp_servers.json`
- `config/mcp_operable_dirs.json`
- `config/scheduled_tasks.json`

Common sensitive variables include (non-exhaustive):

- `GOOGLE_API_KEY`, `MINIMAX_API_KEY`, `DASHSCOPE_API_KEY`, `RUNLOOP_API_KEY`
- `REPORT_DB_HOST`, `REPORT_DB_DATABASE`, `REPORT_DB_USER`, `REPORT_DB_PASS`
- WeChat/WeChat Work credentials

Optional observability switch:

- `WORKMATE_LOG_STDOUT` (`1/true/yes/on`): duplicate Loguru output to stdout while still writing log files

Path safety and write permissions must respect:

- `MCP_READ_ALLOWED_DIRS`
- `MCP_WRITE_ALLOWED_DIRS`
- `OUTPUT_BASE_DIR`

## 6) Coding Standards

### Python

- Target Python `>= 3.12`.
- Follow PEP8 and existing repository style.
- Add type hints for new public functions whenever practical.
- Keep functions focused; split if a function grows beyond clear single responsibility.
- Use structured logging and preserve existing log semantics.
- Fail explicitly with actionable error messages.

### Logging (Loguru)

- Use `loguru` for all new and modified Python logging code. Prefer log statements over ad-hoc `print()` output.
- Standard log levels in this repository are restricted to: `DEBUG`, `INFO`, and `ERROR`.
- Use `DEBUG` for development diagnostics, detailed execution traces, and branch-level internal state.
- Use `INFO` for key business flow milestones and normal runtime lifecycle events.
- Use `ERROR` for failures that affect expected behavior; include actionable context and exception details.
- Keep logs structured and concise, avoid sensitive data in messages, and keep wording stable for downstream parsing/analysis.
- Reference: [Loguru Overview](https://loguru.readthedocs.io/en/stable/overview.html).
- Central setup: [workflow/logging_setup.py](workflow/logging_setup.py) (`init_logging`, `resolve_logs_dir`). [serve.py](serve.py) and [desktop/webview_shell.py](desktop/webview_shell.py) call `init_logging` at startup.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [richsupply/workmate](https://github.com/richsupply/workmate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
