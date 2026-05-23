---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development and Testing
- **Install dependencies**: `uv sync --dev` (from repo root — `pyproject.toml` is at root)
- **Run all tests**: `uv run pytest` (from repo root)
- **Run single test**: `uv run pytest python/tests/core/test_file.py::TestClass::test_method`
- **Linting**: `uv run ruff check`
- **Format**: `uv run ruff format`
- **Fix lint**: `uv run ruff check --fix`
- **Line length**: 120 chars (configured in `pyproject.toml`)

### Benchmarks
- **Langchain benchmarks**: `cd benchmarks/langchain && uv pip install langchain-core langsmith langgraph && uv run pytest bench_*.py`
- **Quick mode** (faster, fewer iterations): default
- **Full mode**: set env `TIMBAL_BENCH_MODE=full`

---

## Repository Layout

```
timbal/
├── python/
│   ├── timbal/               # Main package
│   │   ├── __init__.py       # Top-level exports: Agent, Workflow
│   │   ├── core/
│   │   │   ├── runnable.py   # Base class for all executable components
│   │   │   ├── agent.py      # Agent execution engine
│   │   │   ├── workflow.py   # DAG workflow engine
│   │   │   ├── tool.py       # Tool wrapper
│   │   │   ├── llm_router.py # Multi-provider LLM dispatch
│   │   │   ├── models.py     # Model strings + context window lookup
│   │   │   └── test_model.py # Offline TestModel for testing
│   │   ├── state/
│   │   │   ├── __init__.py   # get_run_context, get_call_id, etc.
│   │   │   ├── context.py    # RunContext definition
│   │   │   └── tracing/
│   │   │       ├── providers/
│   │   │       │   ├── base.py       # TracingProvider ABC + Exporter ABC
│   │   │       │   ├── in_memory.py  # Default in-memory provider
│   │   │       │   ├── jsonl.py      # JSONL file provider
│   │   │       │   └── platform.py   # Timbal platform provider
│   │   │       └── exporters/
│   │   │           └── otel.py       # OTelExporter (fire-and-forget OTLP/HTTP)
│   │   ├── types/
│   │   │   ├── message.py    # Message with role + content list
│   │   │   ├── file.py       # File type with auto content detection
│   │   │   ├── content/      # TextContent, ToolUseContent, FileContent, etc.
│   │   │   └── events/       # StartEvent, DeltaEvent, OutputEvent
│   │   ├── collectors/       # Output processing; TimbalCollector is default
│   │   └── tools/            # Built-in tool library (Bash, Slack, Gmail, etc.)
│   └── tests/
│       └── core/             # Test files mirroring package structure
├── benchmarks/
│   ├── README.md             # General benchmark guide
│   └── langchain/            # Timbal vs LangChain/LangGraph benchmarks
├── planning/                 # In-progress feature plans (gitignored)
└── pyproject.toml            # Root package config + dev deps
```

---

## Core Primitives

### Agent

Autonomous execution unit. An LLM with tools that runs until it decides to stop.

```python
from timbal import Agent

agent = Agent(
    name="my_agent",            # required — used as path in traces
    model="anthropic/claude-sonnet-4-6",  # see Models section
    tools=[my_fn, AnotherTool()],         # functions or Runnable instances
    system_prompt="You are...",           # str or sync/async callable -> str
    max_iter=10,                          # max LLM↔tool iterations (default: 10)
    max_tokens=1024,                      # required for Anthropic models
    output_model=MyPydanticModel,         # structured output via Pydantic
    temperature=0.7,
    model_params={"thinking": {"type": "enabled", "budget_tokens": 2000}},
    tracing_provider=MyProvider,          # see Tracing section
    memory_compaction=compact_tool_results(),
)
```

**Key constructor params:**
- `model` — provider-prefixed string or `TestModel` instance
- `tools` — list of functions, dicts `{"name", "description", "handler"}`, or `Runnable`
- `system_prompt` — str, or a callable (sync/async) that returns str at runtime
- `output_model` — Pydantic model for structured output
- `max_iter` — max LLM→tool→LLM loops before forced stop
- `max_tokens` — required for Anthropic; sets max completion tokens
- `memory_compaction` — strategy or list of strategies; triggers at `memory_compaction_ratio` (default 0.75) of context window
- `tracing_provider` — `TracingProvider` subclass, `None` to disable, or `TRACING_UNSET` (default, auto-detects)
- `default_params` — fixed or callable defaults applied before user kwargs
- `pre_hook` / `post_hook` — parameterless callables; can call `get_run_context()`

---

### Workflow

Explicit DAG execution. Steps run concurrently; dependencies are auto-inferred or explicit.

```python
from timbal import Workflow
from timbal.state import get_run_context

workflow = (
    Workflow(name="my_workflow")
    .step(fetch_data)                           # auto-named "fetch_data"
    .step(                                       # explicit wiring
        process_data,
        data=lambda: get_run_context().step_span("fetch_data").output,
    )
    .step(
        save_result,
        when=lambda: get_run_context().step_span("process_data").output["ok"],
    )
)

result = await workflow.collect(url="https://...")
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timbal-ai/timbal](https://github.com/timbal-ai/timbal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
