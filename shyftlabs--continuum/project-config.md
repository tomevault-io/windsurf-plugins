---
trigger: always_on
description: Continuum Agent Framework — API reference and conventions for the orchestrator source tree
---


# Continuum Agent Framework — Cursor Rule

This is the **Continuum** agentic framework source repository
(Python 3.13). The PyPI package name is `shyftlabs-continuum`; the
import name is `orchestrator`. For development, use editable install
(`pip install -e ".[dev,temporal,eval]"`); downstream consumers
install via `pip install shyftlabs-continuum`.

When generating code, follow the rules below. Treat `AGENTS.md` and
`docs/` as authoritative — never guess at API signatures.

---

## Setup invariants

- **Python 3.13** required. Venv: `python3.13 -m venv .venv`.
- `OPENAI_API_KEY` is required at startup (mem0 uses it for embeddings)
  even when the LLM is Anthropic or Gemini.
- Infra runs in Docker: **Redis on host :6380**, **Qdrant on :6333**.
- All public APIs are **async**. Wrap entrypoints in `asyncio.run(main())`.
- Always `load_dotenv()` at the top of scripts.

---

## Imports cheat sheet

```python
# Agent core
from orchestrator.agent import BaseAgent, AgentRunner
from orchestrator.agent.config import AgentConfig, AgentMemoryConfig, RunnerConfig
from orchestrator.agent.types import (
    Handoff, MemoryScope, RunContext, EventType, AgentResponse,
    HistorySummarizationMode, MergeStrategy, FailStrategy, TerminationType,
)

# Workflow factories — most are re-exported from orchestrator.agent
from orchestrator.agent import (
    create_sequential_agent, create_parallel_agent, create_loop_agent,
    create_reflection_agent, create_planner_agent, create_router_agent,
)
# Debate / scatter / supervised live one level deeper
from orchestrator.agent.workflow import (
    create_debate_agent, create_scatter_agent, create_supervised_agent,
)

# DI container & lifecycle
from orchestrator.core.container import Container, ContainerConfig, get_container
from orchestrator.core.lifecycle import OrchestratorLifecycle, get_lifecycle_manager

# LLM (use directly only when not wrapping in BaseAgent)
from orchestrator.llm import LLMClient, LLMConfig, ChatMessage

# MCP tools
from orchestrator.tools import (
    MCPServerStdio, MCPServerSse, MCPServerStreamableHttp,
    ToolExecutor, MCPUtil,
    ToolContextConfig, ToolContextVariable,
    create_static_tool_filter, ToolFilterContext,
)
from orchestrator.tools.executor import ToolExecutorConfig

# Memory & sessions
from orchestrator.memory import MemoryClient, MemoryConfig
from orchestrator.session import SessionClient

# Observability
from orchestrator.observability import observe, trace_tool, SpanLevel
```

---

## Canonical agent template

```python
import asyncio, os
from dotenv import load_dotenv
from orchestrator.agent import AgentRunner, BaseAgent
from orchestrator.agent.config import AgentMemoryConfig
from orchestrator.agent.types import MemoryScope

load_dotenv()

async def main() -> None:
    agent = BaseAgent(
        name="my-agent",                                # [A-Za-z0-9_-]+
        instructions="You are a helpful assistant.",
        model=os.getenv("DEFAULT_LLM_MODEL", "gpt-4o-mini"),
        memory_config=AgentMemoryConfig(
            search_memories=True, store_memories=True,
            search_scope=MemoryScope.USER, store_scope=MemoryScope.USER,
        ),
    )
    runner = AgentRunner()
    resp = await runner.run(agent, "Hello!", user_id="u1", session_id="s1")
    print(resp.content)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Provider routing rules (model string prefix → provider)

- `gemini/...` or `google/...` → Gemini (OpenAI-compat endpoint)
- `claude/...`, `anthropic/...`, or starts with `claude-` → Anthropic SDK
- everything else (`gpt-*`, `azure/...`, `openai/...`) → OpenAI SDK

**Do not** suggest LiteLLM imports — LiteLLM was removed; the framework
calls provider SDKs directly.

---

## Memory scopes

| Scope | Visibility |
|---|---|
| `MemoryScope.SHARED` | All agents, all users |
| `MemoryScope.USER` | Per user (default) |
| `MemoryScope.AGENT` | Per agent |
| `MemoryScope.RUN` | Single run only |

Without a key, memory init fails. To opt out entirely:
`MEMORY_ENABLED=false` in `.env`, plus `enable_memory=False` if you
construct a `Container` directly.

---

## MCP tooling

```python
mcp = MCPServerStreamableHttp({"url": "https://api.example.com/mcp"})
await mcp.connect()
tools = await MCPUtil.get_function_tools(mcp)
agent = BaseAgent(name="tool-agent", instructions="...", mcp_servers=[mcp])
```

Always `await server.connect()` before use. `ToolExecutor({mcp: None})`
wraps multiple servers under one executor.

---

## Workflow agents (composable, all `BaseAgent` subclasses)

`RouterAgent`, `SequentialAgent`, `ParallelAgent`, `LoopAgent`,
`ReflectionAgent`, `PlannerAgent`, `DebateAgent`, `ScatterAgent`,
`SupervisedSequentialAgent`. Use the `create_*` factories. They nest
freely (a `ParallelAgent` of two `SequentialAgent`s is fine).

```python
pipeline = create_sequential_agent(name="content", agents=[researcher, writer, editor])

debate = create_debate_agent(
    name="debate",
    pro_stance="Argue in favor.",          # strings, not agent instances
    con_stance="Argue against.",
    judge_instructions=None,
)

reflective = create_reflection_agent(
    name="self-improving",
    agent=writer,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shyftlabs/continuum](https://github.com/shyftlabs/continuum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
