---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ConnectOnion is a Python framework for creating AI agents with automatic activity logging, interactive debugging, and multi-agent collaboration. Philosophy: **"Keep simple things simple, make complicated things possible"** - simple 2-line agent creation, but production-ready with trust verification, event system, and plugin architecture.

## Architecture

### Core Components

- **Agent** (`connectonion/agent.py:29`): Main orchestrator with LLM integration, tool execution, event system, and trust verification
- **LLM** (`connectonion/llm.py:11`): Unified abstraction supporting OpenAI, Anthropic, Gemini, and managed keys via factory pattern
- **Tool Executor** (`connectonion/tool_executor.py:24`): Executes tools with xray context injection, timing, error handling, and trace recording
- **Tool Factory** (`connectonion/tool_factory.py`): Converts Python functions to OpenAI-compatible tool schemas automatically
- **Logger** (`connectonion/logger.py`): Unified logging facade (terminal + plain text + YAML sessions) with `quiet` and `log` parameters
- **Console** (`connectonion/console.py`): Low-level terminal output with Rich formatting (used internally by Logger)
- **Events** (`connectonion/events.py:11`): Lifecycle hooks (after_user_input, before_llm, after_llm, before_each_tool, before_tools, after_each_tool, after_tools, on_error, on_complete, on_stop_signal)
- **Trust System** (`connectonion/network/trust/`): Three-level verification (open/careful/strict) with custom policy support
- **XRay Debug** (`connectonion/xray.py:32`): Runtime context injection for interactive debugging with `@xray` decorator

### Key Design Patterns

#### Tool System
- **Function-based (recommended)**: Regular Python functions auto-convert to tools via type hints and docstrings
- **Class-based (legacy)**: Inherit from `Tool` base class with explicit schemas
- Auto-conversion: `create_tool_from_function()` inspects signatures and generates OpenAI schemas

#### Agent Execution Loop (`connectonion/agent.py:input()`)
1. Initialize/extend session with user input
2. Fire `after_user_input` event
3. Loop (max_iterations times):
   - Fire `before_llm` event
   - Call LLM with messages and tool schemas
   - Fire `after_llm` event
   - If tool_calls: execute via `tool_executor.execute_and_record_tools()`
   - Fire `before_tools` event ONCE before ALL tools execute
   - Fire `before_each_tool` and `after_each_tool` events per individual tool
   - Fire `after_tools` event ONCE after ALL tools complete (safe for adding messages)
   - Add results to messages, continue
4. Return final response or iteration limit message

#### Multi-LLM Provider Architecture (`connectonion/llm.py`)
- Factory pattern: `create_llm(model, api_key)` routes to provider classes
- OpenAI format as lingua franca (all providers convert to/from)
- Structured output: Each provider uses native API (OpenAI's `parse()`, Anthropic's forced tool calling, Gemini's `response_schema`)
- Tool calling: Unified `ToolCall` dataclass format across all providers

#### Event System & Plugins (`connectonion/events.py`)
- Wrapper functions tag handlers with `_event_type` attribute
- Plugins are lists of event handlers bundled together
- Handlers receive `agent` instance, can modify `current_session`
- Built-in plugins: reflection, ReAct, image_result_formatter

#### Trust Verification (`connectonion/network/trust/`)
- Three levels: "open" (dev), "careful" (staging), "strict" (prod)
- Custom policies: markdown files or inline text describing verification rules
- Custom agents: Pass your own Agent instance with verification tools
- Environment-based defaults: `CONNECTONION_ENV` sets trust level automatically
- Module structure: `factory.py` (creation), `prompts.py` (level prompts), `tools.py` (verification tools), `trust_agent.py` (TrustAgent class)
- Onboard methods: `invite_code` (verify against configured codes), `payment` (verify via oo-api credit transfer)
- Payment verification: `TrustAgent.verify_payment()` calls oo-api `/api/v1/onboard/verify` to check for recent transfers

#### XRay Debugging (`connectonion/xray.py`)
- `@xray` decorator injects context: `xray.agent`, `xray.task`, `xray.messages`, `xray.iteration`
- `xray.trace()` displays formatted execution history
- `inject_xray_context()` in `tool_executor.py:24` provides runtime context
- Enables interactive debugging with `agent.auto_debug()`

## Development Commands

### Installation
```bash
pip install -r requirements.txt
pip install -e .  # Development mode
```

### Testing
```bash
# Run all tests except real API calls (default)
python -m pytest

# Run specific test categories
python -m pytest tests/unit           # Unit tests (fast, mocked)
python -m pytest tests/integration    # Integration tests (no external APIs)
python -m pytest tests/cli            # CLI tests
python -m pytest tests/e2e            # End-to-end example agent

# Run real API tests (requires keys)
python -m pytest tests/real_api -m real_api

# Run with coverage
python -m pytest --cov=connectonion --cov-report=term-missing

# Run single test file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openonion/connectonion](https://github.com/openonion/connectonion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
