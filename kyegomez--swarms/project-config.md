---
trigger: always_on
description: This file teaches you how to build agents and multi-agent systems with the **Swarms** framework. Read it before writing any code in this repo.
---

# CLAUDE.md — Swarms Framework Guide

This file teaches you how to build agents and multi-agent systems with the **Swarms** framework. Read it before writing any code in this repo.

---

## Installation & Setup

```bash
pip install swarms
```

Set your LLM API key as an environment variable before running:

```bash
export OPENAI_API_KEY="sk-..."        # OpenAI / GPT models
export ANTHROPIC_API_KEY="sk-ant-..." # Claude models
export GROQ_API_KEY="..."             # Groq
# Any provider supported by LiteLLM works
```

All imports come from the top-level `swarms` package:

```python
from swarms import (
    Agent,
    SequentialWorkflow,
    ConcurrentWorkflow,
    AgentRearrange,
    GraphWorkflow,
    SwarmRouter,
    MixtureOfAgents,
    HierarchicalSwarm,
    GroupChat,
    MajorityVoting,
    # ...
)
```

---

## Project Layout

```
swarms/
├── swarms/
│   ├── structs/         # All agent + multi-agent structures (61 files)
│   │   ├── agent.py             # Core Agent class
│   │   ├── conversation.py      # Conversation / memory management
│   │   ├── sequential_workflow.py
│   │   ├── concurrent_workflow.py
│   │   ├── agent_rearrange.py
│   │   ├── graph_workflow.py
│   │   ├── swarm_router.py      # Single-entry-point router
│   │   ├── mixture_of_agents.py
│   │   ├── hiearchical_swarm.py
│   │   ├── groupchat.py
│   │   ├── majority_voting.py
│   │   ├── council_as_judge.py
│   │   ├── debate_with_judge.py
│   │   ├── heavy_swarm.py
│   │   ├── round_robin.py
│   │   ├── planner_worker_swarm.py
│   │   ├── auto_swarm_builder.py
│   │   └── multi_agent_exec.py  # run_agents_concurrently + friends
│   ├── tools/           # Tool utilities, MCP, schema conversion
│   └── utils/           # Logging, formatting helpers
├── examples/            # 586 runnable examples
│   ├── single_agent/
│   ├── multi_agent/
│   ├── tools/
│   └── guides/
└── v12_examples/        # New v12 feature examples
```

Look in `examples/` first before writing new code — there is almost certainly an existing example close to what you need.

---

## Core Primitive: Agent

`Agent` is the single building block everything else composes. All multi-agent structures wrap one or more `Agent` instances.

### Minimal agent

```python
from swarms import Agent

agent = Agent(
    agent_name="Analyst",
    model_name="gpt-5.4",
    max_loops=1,
)

result = agent.run("Summarise the current state of LLM research.")
print(result)
```

### Key constructor parameters

| Parameter | Type | Default | Purpose |
|---|---|---|---|
| `agent_name` | str | `"swarm-worker-01"` | Unique name — used for memory file paths |
| `agent_description` | str | generic | Shown to orchestrators for routing |
| `system_prompt` | str | built-in | The agent's persona / instructions |
| `model_name` | str | `"gpt-5.4"` | Any LiteLLM model string |
| `max_loops` | int \| `"auto"` | `1` | Loops before returning; `"auto"` = autonomous until done |
| `tools` | list[Callable] | `None` | Python functions the agent can call |
| `streaming_on` | bool | `False` | Stream tokens to stdout |
| `interactive` | bool | `False` | REPL mode — prompt user for input each loop |
| `context_length` | int | `None` | Token budget; triggers compression at 90 % |
| `context_compression` | bool | `True` | Auto-summarise when near context limit (v12) |
| `persistent_memory` | bool | `True` | Read/write MEMORY.md across restarts (v12) |
| `temperature` | float | `0.5` | Sampling temperature |
| `max_tokens` | int | `4096` | Max tokens per LLM call |
| `reasoning_effort` | str | `None` | `"low"`, `"medium"`, `"high"` for reasoning models |
| `thinking_tokens` | int | `None` | Extended thinking budget (Claude) |
| `output_type` | str | `"str-all-except-first"` | How to format returned output |
| `mcp_url` | str | `None` | MCP server URL to load tools from |
| `handoffs` | list | `None` | Agents this agent can hand off to |
| `plan_enabled` | bool | `False` | Generate a plan before execution |
| `autosave` | bool | `False` | Save agent state to disk after each run |

### Autonomous loop (`max_loops="auto"`)

When `max_loops="auto"` the agent runs a plan→execute→reflect loop until it decides it is done. It automatically gets access to:
- A `think` tool (disabled when `thinking_tokens` is set)
- A `grep` tool for searching files (v12)
- Bash / file tools if configured

```python
agent = Agent(
    agent_name="Researcher",
    model_name="gpt-5.4",
    max_loops="auto",
    interactive=False,
)
result = agent.run("Research the top 5 vector databases and compare them.")
```

### Model names

Use any LiteLLM-compatible string:

```python
# OpenAI
model_name="gpt-5.4"
model_name="gpt-5.4-mini"
model_name="o3"

# Anthropic
model_name="claude-opus-4-7-20251001"
model_name="claude-sonnet-4-6"
model_name="claude-haiku-4-5-20251001"

# Groq
model_name="groq/llama-3.3-70b-versatile"

# Google
model_name="gemini/gemini-2.5-pro"
```

### Running with images

```python
result = agent.run(
    task="Describe what you see in this chart.",
    img="path/to/chart.png",   # or base64 string or URL
)
```

---

## Memory & Persistence (v12)

### `persistent_memory=True` (default)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyegomez/swarms](https://github.com/kyegomez/swarms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
