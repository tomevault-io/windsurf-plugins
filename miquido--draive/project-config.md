---
trigger: always_on
description: `Agent` is Draive's lightweight abstraction for building reusable async workers that:
---

# Agents

`Agent` is Draive's lightweight abstraction for building reusable async workers that:

- accept typed multimodal input,
- stream visible output chunks and `ProcessingEvent`s,
- preserve conversation thread and metadata through `ctx.scope(...)`, and
- can be exposed to other agents as tools through `AgentsGroup`.

The package is intentionally small. It builds on top of existing Draive primitives:

- `Step` for custom execution pipelines,
- `GenerativeModel` for model-backed execution,
- `Toolbox` for tool handling,
- `MultimodalContent` for input and output transport.

## Runtime Model

The agents API is intentionally built as a thin layer over existing Draive runtime abstractions.

- `AgentIdentity` describes the agent instance: `uri`, `name`, `description`, `meta`.
- `AgentMessage` is the fully prepared input payload: `thread`, `created`, `content`, `meta`.
- `AgentThread` is the scoped runtime state propagated through `ctx.scope(...)`.
- `AgentExecuting` is the executor protocol:
    `AgentMessage -> AsyncIterable[MultimodalContentPart | ProcessingEvent]`.

In other words, `Agent` itself is not a stateful conversation object. It is an immutable wrapper
that runs an executor inside a scoped agent context and streams output.

## 1. Build An Agent From `Step`s

Use `Agent.steps(...)` when you already have a `Step` pipeline and want to expose it as an
agent.

```python
from collections.abc import AsyncIterable

from draive import Agent, ProcessingEvent
from draive.multimodal import TextContent
from draive.steps import Step, StepState


async def execute(
    state: StepState,
) -> AsyncIterable[ProcessingEvent | TextContent | StepState]:
    yield ProcessingEvent.of("progress", "Analyzing request...")
    yield TextContent.of("Done")
    yield state


worker: Agent = Agent.steps(
    Step(execute),
    agent="worker",
    description="Handles a small processing task",
)
```

Call the agent inside a context scope and consume the stream.

```python
from collections.abc import AsyncIterable

from draive import ctx
from draive.multimodal import MultimodalContentPart
from draive.utils import ProcessingEvent


async with ctx.scope("agents.step"):
    stream: AsyncIterable[MultimodalContentPart | ProcessingEvent] = worker.call(
        input="Please help"
    )
    async for chunk in stream:
        print(chunk)
```

If you need lower-level control, build `AgentMessage` yourself and call `respond(...)` directly.

```python
from collections.abc import AsyncIterable

from draive import AgentMessage
from draive.multimodal import MultimodalContentPart
from draive.utils import ProcessingEvent


message: AgentMessage = AgentMessage.of("Please help")

async with ctx.scope("agents.respond"):
    stream: AsyncIterable[MultimodalContentPart | ProcessingEvent] = worker.respond(
        message
    )
    async for chunk in stream:
        print(chunk)
```

### What `steps(...)` Does

- Prepends the incoming agent message into step state with `Step.appending_input(...)`.
- Executes your step pipeline.
- Filters out `ModelReasoningChunk`.
- Treats leaked `ModelToolRequest` and `ModelToolResponse` chunks as an internal contract violation.
- Streams only user-visible content and `ProcessingEvent`s.

This makes step-backed agents a good fit when you want deterministic orchestration and typed state
updates, but a clean public output stream.

One important implication: if your step emits reasoning, callers of the agent will not see it. If
your step emits tool protocol chunks, `Agent.steps(...)` will raise `AssertionError` in debug mode
instead of exposing them to callers. `Agent.steps(...)` is intentionally a public-facing wrapper
over a more verbose internal step stream.

## 2. Build A Generative Model-Backed Agent

Use `Agent.generative(...)` when the agent should directly call the configured
`GenerativeModel.completion(...)`.

```python
from collections.abc import AsyncIterable

from draive import Agent, ctx, load_env, tool
from draive.multimodal import MultimodalContentPart
from draive.openai import OpenAI, OpenAIResponsesConfig
from draive.utils import ProcessingEvent


load_env()


@tool(description="Return current system status")
async def system_status() -> str:
    return "All systems operational"


assistant: Agent = Agent.generative(
    agent="support",
    description="Answers product support questions",
    instructions="You are a concise support assistant. Use tools when useful.",
    tools=[system_status],
)


async with ctx.scope(
    "agents.generative",
    OpenAIResponsesConfig(model="gpt-5-mini"),
    disposables=(OpenAI(),),
):
    stream: AsyncIterable[MultimodalContentPart | ProcessingEvent] = assistant.call(
        input="Check the current system status"
    )
    async for chunk in stream:
        print(chunk)
```

### How The Generative Loop Works

For each call, the agent:

1. converts the incoming message into `ModelInput`,
1. calls `GenerativeModel.completion(...)`,
1. collects any `ModelToolRequest`s,
1. executes them through `Toolbox.handle(...)`,
1. appends `ModelToolResponse`s back into model context,
1. repeats until the model produces a final answer.

If a tool uses `handling="output"`, the tool can stream visible output directly and terminate the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miquido/draive](https://github.com/miquido/draive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
