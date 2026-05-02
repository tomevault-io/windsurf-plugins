---
trigger: always_on
description: This file provides context for AI coding agents (Claude Code, Copilot, Cursor, etc.) working on the Agentspan SDK.
---

# AGENTS.md — Guide for AI Agents Working on This Codebase

This file provides context for AI coding agents (Claude Code, Copilot, Cursor, etc.) working on the Agentspan SDK.

## Project Overview

The `agentspan` Python SDK compiles Python `Agent` definitions into durable [Conductor](https://github.com/conductor-oss/conductor) executions. Agents survive process crashes, tools scale as distributed workers, and human-in-the-loop approvals can pause for days.

**Package name (PyPI):** `agentspan`
**npm package:** `@agentspan-ai/agentspan`
**Import path:** `from agentspan.agents import ...`
**Python:** 3.10+
**License:** MIT

## Architecture

### Core Design Principles

1. **Everything is an Agent.** One class for single agents, multi-agent teams, and nested hierarchies. No Team, Network, or Swarm classes.
2. **Server-first execution.** Tools execute as distributed Conductor tasks. The agent survives process crashes.
3. **Compile, don't interpret.** Agent definitions are compiled into static Conductor workflow JSON at registration time.
4. **Zero config for simple cases.** `Agent + tool + run` works in 5 lines.
5. **Conductor-native.** Every SDK concept maps directly to a Conductor primitive.

### Compilation Pipeline

```
Agent(Python) → AgentCompiler.compile() → ConductorWorkflow(JSON) → execute on server
```

When `run(agent, prompt)` is called:
1. Agent is compiled into a Conductor workflow definition
2. Worker processes are started for `@tool` functions
3. Agent is executed on the Conductor server
4. Result is extracted and returned as `AgentResult`

### Key Source Files

| File | Purpose |
|---|---|
| `src/agentspan/agents/agent.py` | `Agent` class — the single orchestration primitive |
| `src/agentspan/agents/tool.py` | `@tool` decorator, `ToolDef`, `http_tool()`, `mcp_tool()` |
| `src/agentspan/agents/run.py` | Top-level `run()`, `start()`, `stream()`, `run_async()`, `plan()` with singleton runtime |
| `src/agentspan/agents/result.py` | `AgentResult`, `AgentHandle`, `AgentStatus`, `AgentEvent`, `EventType` |
| `src/agentspan/agents/guardrail.py` | `Guardrail`, `GuardrailResult`, `RegexGuardrail`, `LLMGuardrail` |
| `src/agentspan/agents/memory.py` | `ConversationMemory` — session message history |
| `src/agentspan/agents/semantic_memory.py` | `SemanticMemory`, `MemoryStore`, `MemoryEntry` — long-term memory |
| `src/agentspan/agents/termination.py` | `TerminationCondition` and composable subclasses (`&`, `|` operators) |
| `src/agentspan/agents/handoff.py` | `HandoffCondition`, `OnToolResult`, `OnTextMention`, `OnCondition` |
| `src/agentspan/agents/code_executor.py` | `CodeExecutor` — Local, Docker, Jupyter, Serverless |
| `src/agentspan/agents/ext.py` | `UserProxyAgent`, `GPTAssistantAgent` |
| `src/agentspan/agents/tracing.py` | Optional OpenTelemetry integration |
| `src/agentspan/agents/__init__.py` | Public API surface — all exports |
| `src/agentspan/agents/compiler/agent_compiler.py` | Single agent compilation (DoWhile loops, tool dispatch) |
| `src/agentspan/agents/compiler/multi_agent_compiler.py` | Multi-agent strategies (handoff, sequential, parallel, router) |
| `src/agentspan/agents/compiler/tool_compiler.py` | `@tool` → TaskDef + ToolSpec + dispatch registration |
| `src/agentspan/agents/compiler/_dispatch.py` | Universal dispatch worker (fuzzy parsing, circuit breaker) |
| `src/agentspan/agents/runtime/runtime.py` | `AgentRuntime` — compile + execute + stream |
| `src/agentspan/agents/runtime/worker_manager.py` | Auto-register `@tool` as Conductor workers |
| `src/agentspan/agents/runtime/config.py` | `AgentConfig` — environment variable configuration |
| `src/agentspan/agents/_internal/model_parser.py` | Parse `"provider/model"` strings |
| `src/agentspan/agents/_internal/schema_utils.py` | JSON Schema generation from type hints |

### Conductor Primitive Mapping

| SDK Concept | Conductor Primitive |
|---|---|
| `Agent` | `ConductorWorkflow` |
| `@tool` function | Task definition + `@worker_task` |
| `http_tool` | `HttpTask` (server-side) |
| `mcp_tool` | `ListMcpTools` + `CallMcpTool` |
| Agentic loop | `DoWhileTask` |
| LLM call | `LlmChatComplete` (system task) |
| Handoff | `InlineSubWorkflowTask` |
| Sequential | Chain of `SubWorkflowTask` |
| Parallel | `ForkTask` + `JoinTask` |
| Human approval | `WaitTask` |
| Conversation state | `workflow.variables` |

## Coding Conventions

### Style

- **Linter:** ruff (`target-version = "py310"`, `line-length = 100`)
- **Type checker:** mypy (`python_version = "3.10"`, `ignore_missing_imports = true`)
- **Imports:** isort via ruff (`"I"` rule)
- **Python target:** 3.10+ (use `from __future__ import annotations` for newer typing syntax)

### Module-Level Patterns

- Every module uses `logging.getLogger("agentspan.agents.xxx")` for structured logging
- The dispatch worker (`_dispatch.py`) deliberately does NOT use `from __future__ import annotations` because Conductor's worker framework needs real type objects for parameter resolution
- The dispatch worker uses `object` type annotations (not `dict`/`list`) to avoid Conductor's `convert_from_dict_or_list()` issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentspan-ai/agentspan](https://github.com/agentspan-ai/agentspan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
