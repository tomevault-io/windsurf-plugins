---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

TraceClaw is an enterprise-grade transparent AI agent framework built on LangGraph. It gives users full observability into what their AI agent is doing — every LLM call, tool invocation, and system action is audited via JSONL logs and a Rich-based monitor terminal.

## Commands

```bash
# Install in editable mode (also installs all dependencies)
pip install -e .

# Alternative: install deps only (no package install)
pip install -r requirements.txt

# Run the interactive agent
traceclaw run

# Run the configuration wizard (sets up .env)
traceclaw config

# Run the monitor terminal (in a separate window — tail logs)
traceclaw monitor

# Run all tests
python -m pytest tests/ -v

# Run a single test file
python tests/test_builtins.py

# Run the two-phase skills experiment
python -c "from tests.test_two_phase_skills import run_tests; run_tests()"

# Run the lazy-loading benchmark
python examples/benchmark_lazy_loading.py
```

## Architecture

The system is a **LangGraph StateGraph** with two nodes — `agent` and `tools` — running in a loop (agent → tools → agent until the agent decides to reply directly).

### Entry points

Three CLI commands are registered via `setup.py` → `entry/cli.py` (Typer app):
- `traceclaw run` → `entry/main.py` — full interactive agent with Prompt Toolkit UI
- `traceclaw config` → `entry/cli.py:config_wizard` — interactive .env setup wizard
- `traceclaw monitor` → `entry/monitor.py` — live JSONL log viewer using Rich panels

### Core modules

| Module | Role |
|--------|------|
| `traceclaw/core/agent.py` | Builds the LangGraph StateGraph. Constructs the system prompt from user profile + context summary, binds tools, invokes the LLM, audits every step. The `agent_node` function is the central decision loop. |
| `traceclaw/core/config.py` | Resolves all directory paths (`WORKSPACE_DIR`, `OFFICE_DIR`, `SKILLS_DIR`, `DB_PATH`, etc.) from `TRACECLAW_WORKSPACE` env var or defaults to `<project>/workspace/`. Also creates dirs on import. |
| `traceclaw/core/provider.py` | LLM provider factory — maps provider name strings to LangChain chat models (ChatOpenAI for OpenAI/aliyun/z.ai/tencent, ChatAnthropic, ChatOllama). Includes fallback base URLs for popular Chinese providers. |
| `traceclaw/core/context.py` | Defines `AgentState` (messages + summary) and `trim_context_messages()` — when total turns exceed `trigger_turns` (40), older turns are discarded and summarized, keeping the most recent `keep_turns` (10). Groups messages by user turn (HumanMessage → next HumanMessage). |
| `traceclaw/core/skill_loader.py` | `LazySkillLoader` class — scans `workspace/office/skills/` for folders containing `SKILL.md`, extracts name/description from frontmatter, and creates LangChain `StructuredTool` wrappers. Uses LRU caching (max 50). Tools enforce a **two-phase call protocol**: `mode='help'` first (returns the SKILL.md content), then `mode='run'` with the assembled command. |
| `traceclaw/core/logger.py` | Singleton `JSONLEventLogger` — background thread writes structured JSONL events to `logs/<thread_id>.jsonl`. Five event types: `llm_input`, `tool_call`, `tool_result`, `ai_message`, `system_action`. |
| `traceclaw/core/bus.py` | Global `asyncio.Queue` singleton — the single input channel for the entire system. User keyboard input and heartbeat task triggers both `put()` onto `task_queue`; the `agent_worker` coroutine blocks on `get()` and processes messages serially. This design avoids multi-user concurrency issues at the cost of no parallel request handling (correct tradeoff for a personal agent). |
| `traceclaw/core/heartbeat.py` | `pacemaker_loop` — async coroutine that polls `workspace/tasks.json` every 10s. When a task's `target_time` ≤ now, it calls `emit_task()` (from `bus.py`) to push a message onto the shared `task_queue`, then computes the next occurrence for recurring tasks (hourly/daily/weekly/monthly). |

### Tools layer

- `traceclaw/core/tools/base.py` — `TraceClawBaseTool` ABC (class-based tool) and `traceclaw_tool` alias for LangChain's `@tool` decorator.
- `traceclaw/core/tools/builtins.py` — 12 built-in tools compiled into `BUILTIN_TOOLS` list. Includes time queries, calculator, user profile management, task scheduling (CRUD on `workspace/tasks.json`), and the four sandbox tools. Uses a `threading.Lock` for task file writes.
- `traceclaw/core/tools/sandbox_tools.py` — File I/O and shell execution confined to `OFFICE_DIR`. `_get_safe_path()` enforces that all resolved absolute paths start with the office directory. `execute_office_shell` has five regex-based dangerous-pattern checks preventing path traversal, absolute paths, and drive-letter escapes on Windows. Commands get a 60-second timeout.

### Memory system (dual-watermark)

- **Long-term**: `workspace/memory/user_profile.md` — manually or tool-edited Markdown file storing user preferences, injected into the system prompt.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fubenshuo/TraceClaw](https://github.com/fubenshuo/TraceClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
