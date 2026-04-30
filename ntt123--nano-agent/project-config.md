---
trigger: always_on
description: **nano_agent** is a Python 3.12 library for building AI agents around a
---

# Guidance for working with this repository

## Project Overview

**nano_agent** is a Python 3.12 library for building AI agents around a
functional, immutable conversation DAG.

- **Immutable graph model**: `DAG` and `Node` operations return new instances.
- **Node-based history**: system prompts, tool definitions, messages, tool
  executions, sub-graphs, and stop reasons are graph nodes.
- **Typed tools**: tools are dataclasses with typed `__call__` input dataclasses;
  schemas are inferred from annotations and `Desc`/`Field` metadata.
- **Provider-neutral loop**: providers implement `APIProtocol`; `executor.run`
  sends the DAG, executes tool calls, merges tool-result branches, and appends a
  stop reason.
- **Applications**: `cli/` implements the `nano-cli` terminal assistant, and
  `bot/` implements Discord and Slack bot frontends.

## Repository Layout

- `nano_agent/`: core library.
  - `dag.py`: immutable `Node` and `DAG` graph primitives, serialization,
    sub-graph conversion, console rendering.
  - `data_structures.py`: content blocks, messages, responses, usage, graph node
    payload types, and parsing helpers.
  - `executor.py`: generic agent loop with transient retry, tool execution,
    cancellation handling, usage accounting, and stop-reason nodes.
  - `execution_context.py`: sub-agent context and `run_sub_agent`.
  - `tools/`: built-in tools and `Tool`/`SubAgentTool` base classes.
  - `providers/`: Claude, Claude Code OAuth, Gemini, OpenAI, Fireworks, and
    Codex/ChatGPT OAuth clients plus auth helpers and cost tracking.
- `cli/`: message-list terminal UI, command router, footer input system, session
  persistence, and DAG-to-UI mapping.
- `bot/`: Discord and Slack frontends, shared queue/state management, shared
  channel worker, and platform-specific tools.
- `tests/`: unit tests for core, providers, tools, CLI UI pieces, and bot logic.
- `e2e/`: real-provider end-to-end tests; do not run as part of ordinary checks.
- `examples/`: runnable examples and generated sub-agent graph artifacts.
- `scripts/`: graph viewers and rendering helpers.

## Development Commands

```bash
# Install/update dependencies
uv sync
uv sync --extra bot
uv sync --extra tmux

# Unit tests
uv run pytest
uv run pytest --ignore=e2e
uv run pytest tests/test_dag.py -v
uv run pytest bot/test_bot.py -v

# End-to-end tests; these make real API calls
uv run python e2e/run_all.py
uv run python e2e/test_executor_cancellation.py

# Type checking and formatting
uv run mypy .
uv run black .
uv run isort --profile black .
uv run pre-commit run --all-files

# Examples
uv run python examples/hello_world.py
uv run python examples/simple_tool.py
uv run python examples/sub_agent.py
```

Pre-commit runs Black, isort, mypy with `tests/` and `e2e/` excluded, and
`uv run pytest --ignore=e2e`.

## Core Patterns

```python
from nano_agent import DAG, BashTool, ClaudeAPI, run

dag = (
    DAG()
    .system("You are helpful.")
    .tools(BashTool())
    .user("What is the current date?")
)
dag = await run(ClaudeAPI(), dag)
```

Important graph rules:

- Do not mutate `DAG`, `Node`, or content-block instances in place.
- Use fluent DAG methods (`system`, `tools`, `user`, `assistant`, `tool_result`,
  `sub_graph`) or internal helpers only where existing code already does so.
- `DAG._tools` stores live tool instances for execution; serialized graph data
  stores tool definitions for provider calls and visualization.
- Tool calls execute from one assistant head, create one branch per result, and
  merge through a synthetic user message containing all `ToolResultContent`
  blocks.
- Cancellation must leave every started or pending tool call with a matching
  tool result so provider conversations remain valid.

## Tool Patterns

Tools live under `nano_agent/tools/` and usually follow this shape:

```python
from dataclasses import dataclass
from typing import Annotated

from nano_agent import TextContent
from nano_agent.tools.base import Desc, Tool

@dataclass
class CalculatorInput:
    expr: Annotated[str, Desc("Expression to evaluate")]

@dataclass
class CalculatorTool(Tool):
    name: str = "Calculator"
    description: str = "Evaluate a small expression"

    async def __call__(self, input: CalculatorInput) -> TextContent:
        return TextContent(text=str(eval(input.expr)))
```

- Prefer `Annotated[T, Desc("...")]` for schema descriptions.
- Call tools through `tool.execute(...)` in framework code so raw JSON inputs are
  converted, output truncation is applied, and sub-agent context is passed.
- Return `TextContent`, `list[TextContent]`, or `ToolResult`.
- Configure large-output behavior with `_truncation_config`.
- Add external command checks with `_required_commands` when a tool depends on a
  CLI binary.
- `TmuxTool` is available but is not in `get_default_tools()`; it requires the
  `tmux` extra and the `tmux` binary.

## Sub-Agents

Use `SubAgentTool` when a tool needs to run another agent and preserve the child
graph:

```python
from dataclasses import dataclass
from typing import Annotated

from nano_agent import ExecutionContext, ReadTool, SubAgentTool, TextContent
from nano_agent.tools.base import Desc, ToolResult

@dataclass
class AuditInput:
    file_path: Annotated[str, Desc("Path to audit")]

@dataclass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NTT123/nano-agent](https://github.com/NTT123/nano-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
