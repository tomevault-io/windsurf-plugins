---
trigger: always_on
description: This file provides guidance to Codex when working with code in this repository. It is a Codex-facing mirror of `CLAUDE.md` — the two should stay in lockstep.
---

# AGENTS.md

This file provides guidance to Codex when working with code in this repository. It is a Codex-facing mirror of `CLAUDE.md` — the two should stay in lockstep.

## Build & Development Commands

```bash
# Install all dependencies (extras + dev — dev is needed for pytest/ruff/mypy)
uv sync --all-extras --dev

# Run tests
uv run pytest

# Run single test file
uv run pytest tests/test_trace.py

# Run tests with coverage
uv run pytest --cov=arcana

# Lint
uv run ruff check .

# Type check (strict mode)
uv run mypy src/

# Run integration tests (requires API keys via env vars or .env)
uv run pytest tests/integration/ -v

# Run demo
uv run python examples/demo_trace.py

# Build docs
uv sync --extra docs && uv run mkdocs build

# Serve docs locally
uv run mkdocs serve
```

## Architecture Overview

Arcana is an Agent Platform following a "contracts-first" design. All modules define Pydantic schemas before implementation, enabling future language migrations (Go/Rust) without changing upper-layer logic.

### V2 Architecture (current)

The V2 engine centers on `ConversationAgent` -- no Policy, no Reducer, just LLM turns. Each turn produces `TurnFacts` (raw LLM output) and `TurnAssessment` (runtime interpretation, including thinking-informed confidence adjustment), kept visibly separate.

Key capabilities beyond basic turn loop:
- **Parallel tool execution**: multiple tool calls in one turn run concurrently via `asyncio.gather` with order-preserving results
- **`ask_user` built-in tool**: LLM can ask clarifying questions mid-execution; intercepted at runtime level (bypasses ToolGateway); graceful fallback when no handler provided
- **Prompt caching**: transparent, provider-level -- Anthropic `cache_control` tags on system/tools, OpenAI `cached_tokens` tracked
- **Thinking-informed assessment**: `_assess_turn` analyzes extended thinking blocks for uncertainty/verification/incomplete signals, adjusts confidence
- **Structured output**: `response_format` passes a Pydantic model's JSON Schema to the provider; tools remain available and coexist with structured output
- **Multimodal input**: `images` parameter accepts URLs, file paths, data URIs; auto-converts between OpenAI and Anthropic content block formats
- **Fidelity-graded context compression**: `WorkingSetBuilder.abuild_conversation_context()` compresses history using 4 fidelity levels (L0 original → L3 dropped) based on relevance scoring; falls back to LLM summarization or aggressive truncation
- **Multi-turn chat**: `runtime.chat()` returns a `ChatSession` that delegates to `ConversationAgent` internally, gaining all V2 features (ask_user, lazy tools, diagnostics, fidelity compression, thinking assessment)
- **Sequential pipeline**: `runtime.chain()` runs a list of `ChainStep`s sequentially, automatically passing each step's output as context to the next
- **Context passing**: `runtime.run(context=...)` injects additional context (dict or string) into the agent's goal as a `<context>` block

```
Request -> Intent Router (routing/)
            -> Direct Answer (1 LLM call)
            -> ConversationAgent (runtime/conversation.py)
                 LLM Turn -> TurnFacts -> TurnAssessment -> State
                 Runtime OS: Budget | Trace | Tools | Diagnostics | ask_user

Multi-turn:    runtime.chat() -> ChatSession -> send() / stream()
Multi-agent:   runtime.collaborate() -> AgentPool (user controls who speaks
               and when; runtime provides shared infra). runtime.team() was
               removed in v1.0.0.
Pipeline:      runtime.chain([ChainStep, ...]) -> sequential run() with auto context
Batch:         runtime.run_batch([tasks], concurrency=...) -> list[BatchResult]

V1 path (still compatible):
            -> Agent + AdaptivePolicy (runtime/agent.py)
```

### Layer Structure

```
+-------------------------------------------------+
|              Application (Agents)                |
|   runtime/conversation.py  (V2 ConversationAgent)|
|   runtime/agent.py         (V1 Agent)            |
|-------------------------------------------------|
|  Routing | Gateway | Tool Gateway | Context      |  <- Platform Services
|  Eval    | Memory  | Streaming    | Diagnosis    |
|-------------------------------------------------|
|           Contracts (Pydantic Schemas)           |  <- Data Models
|   turn, routing, context, diagnosis, llm, tool   |
|-------------------------------------------------|
|              Trace (JSONL Audit Log)             |  <- Observability
+-------------------------------------------------+
```

### Core Modules

**contracts/** - All data models:
- `turn.py`: `TurnFacts`, `TurnAssessment` -- the V2 separation principle
- `routing.py`: `RoutingDecision`, `IntentCategory` -- intent classification
- `context.py`: `ContextBlock`, `TokenBudget`, `ContextDecision` -- working set context
- `diagnosis.py`: `ErrorDiagnosis`, `ErrorCategory` -- structured error recovery
- `llm.py`: `LLMRequest`, `LLMResponse`, `ModelConfig`, `ContentBlock` -- unified LLM interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tyxben/arcana](https://github.com/tyxben/arcana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
