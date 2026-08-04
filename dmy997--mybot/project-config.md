---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

mybot is a multi-provider AI agent framework with plugin-style agents, streaming output, long-term memory, HTTP/WS API, and extensible chat channels (WeChat). Designed to work with any OpenAI-compatible API endpoint. 1037 tests, 1033 passing.

## Development Setup

```bash
pip install -e ".[dev,server]"
cp .env.example .env   # then fill in your keys
```

## Build & Test

```bash
ruff check .           # lint
pytest                 # all 1037 tests (pytest-asyncio, asyncio_mode = "auto")
pytest test/core/test_middleware.py -v   # single file
pytest test/providers/test_openai_compatible_provider.py::TestParseDict::test_dict_with_choices -v
```

## Running

```bash
mybot                  # interactive CLI (core.orchestrator:main)
mybot-server           # HTTP/WS server (core.server:main), then open http://127.0.0.1:8080
```

## Package Layout (flat, no `mybot/` subdirectory)

- `providers/` — LLM backend abstraction (`LLMProvider` base, `OpenAICompatibleProvider`, retry logic, error types, factory)
- `core/` — Orchestrator (composed from 4 mixins: MCPServices, ToolRegistry, SessionLifecycle, IdleCompression), Dispatcher, AgentCore (runner), Agent base class, Middleware chain, SkillsLoader, HTTP/WS server
- `agents/` — ReAct Agent (single-pass) + PlanSolve Agent (two-phase). Auto-discovered via `discover_agents()`
- `evals/` — Agent evaluation system (custom YAML tasks + BFCL/GAIA benchmarks)
- `context/` — ContextManager (unified, no mixins): build_messages, compress, semantic filter for tools/skills. SessionManager (JSON persistence with hard-cap pruning + TTL expiry), CompactionService, MemoryService, SemanticFilter (embedding-based cosine similarity ranking)
- `tools/` — bash, file R/W, grep, webfetch, websearch, memory CRUD, subagent, `schedule_task` (create/list/cancel periodic tasks), ToolRegistry, ToolGuard security
- `services/` — `CronScheduler` (self-driven timer, cron-expression + interval jobs) + `ScheduledTaskService` (chat-created push tasks + system side-effect tasks like Xiaohongshu) + `HitlService` / `HitlMiddleware` (human-in-the-loop confirmation). See `docs/scheduled-tasks.md` and `docs/hitl.md`
- `memory/` — Long-term file-based memory (MemoryStore, Consolidator + Dream pipeline)
- `observability/` — Structured logging (loguru), Prometheus-style metrics, span tracing, rich CLI display
- `config/` — `.env` auto-loading, typed `Config` class
- `utils/` — Jinja2 template rendering (`render_template()`), shared embedding model (`EmbeddingModel` singleton for semantic similarity)
- `prompt_templates/` — 14 agent prompt templates (Jinja2 `.md`)
- `skills/` — 22 pre-built skill directories (art, pdf, xlsx, xiaohongshu, frontend-design, etc.). SkillsLoader auto-discovers and loads them
- `server_web/` — Single `index.html` for the browser chat UI

## Request Flow

```
HTTP/WS or CLI → Orchestrator → ContextManager.build_messages()
                                   ├─ repair interrupted session
                                   ├─ assemble system prompt (base + memory + skills (triggered+semantic) + tools (similarity-ranked) + history summaries)
                                   ├─ load session history (cursor-based, 100-msg cap)
                                   └─ token-budget check → compress if needed
                → Dispatcher.resolve()
                    ├─ Layer 1: explicit commands (/react, /plan)
                    ├─ Layer 2: keyword heuristics (multi-step → plan_solve)
                    ├─ Layer 3: LLM classification (optional, cheap model)
                    └─ Layer 4: default (react)
                → Agent.run(AgentInput) → AgentCore.run()
                    └─ loop: LLM call → tool calls (parallel + serial) → feed results back
                → ContextManager.save_exchange() → persist to disk
```

## Key Abstractions & Patterns

**`LLMProvider` → `OpenAICompatibleProvider`** (`providers/base.py`, `providers/openai_compatible_provider.py`):
- Abstract base: `async chat(messages, tools, model, max_tokens, temperature) -> LLMResponse`
- Also provides `safe_chat()`, `chat_with_retry()`, `chat_stream()` (true SSE with delta callbacks), `chat_stream_with_retry()`
- Lazy `AsyncOpenAI` client init protected by async lock
- OpenRouter auto-detection: sets referer/session-affinity headers when `name="openrouter"` or URL contains "openrouter"

**`AgentCore`** (`core/runner.py`): The shared execution loop used by ALL agent paradigms. Calls LLM in a loop, executes tool calls, feeds results back. Handles:
- Parallel vs serial tool execution (`asyncio.gather` for parallel-safe tools)
- Context compaction (7-step pipeline on copies: remove orphans → fill missing → summarize old → truncate long → token budget → repair)
- LLM error recovery (context_length → compact & retry; content_filter → append compliance hint & retry)
- Stall detection at 50+ steps
- Middleware hooks at agent start/step/end, LLM call, tool execution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmy997/mybot](https://github.com/dmy997/mybot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
