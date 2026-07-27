---
trigger: always_on
description: Agents are the core building block in Datapizza AI.
---

# Build your first agent

Agents are the core building block in Datapizza AI.

An `Agent` combines:

- a `name`
- a `system_prompt`
- a `client`
- optional tools, memory, hooks, structured output, and handoffs

Use this page to get an agent running quickly, then add the capabilities you need.

## Create your first agent

Start with the smallest useful setup.

```python
from datapizza.agents import Agent
from datapizza.clients.openai import OpenAIClient

agent = Agent(
    name="assistant",
    system_prompt="You are a helpful assistant.",
    client=OpenAIClient(api_key="YOUR_API_KEY", model="gpt-4o-mini"),
)
```

The only required pieces are:

- `name`: a human-readable name for the agent
- `system_prompt`: the instructions the model follows
- `client`: the model provider implementation

## Run your first agent

Call `run(...)` and read the final answer from `result.text`.

```python
result = agent.run("Write a one-line welcome message for a new user.")
print(result.text)
```

`run(...)` returns a `StepResult`, not a plain string.

The most useful properties are:

- `result.text`: the final text answer
- `result.tools_used`: tools called in that step
- `result.structured_data`: parsed structured output when `output_cls` is set
- `result.usage`: token usage aggregated for the run

## Give your agent tools

Tools let the agent fetch data or perform actions.

```python
from datapizza.agents import Agent
from datapizza.clients.openai import OpenAIClient
from datapizza.tools import tool


@tool
def get_weather(location: str, when: str) -> str:
    """Return weather information for a location and time."""
    return "25 C"


agent = Agent(
    name="weather_agent",
    system_prompt="You help users with weather questions.",
    client=OpenAIClient(api_key="YOUR_API_KEY", model="gpt-4o-mini"),
    tools=[get_weather],
)

result = agent.run("What's the weather tomorrow in Milan?")
print(result.text)
```

### Control tool use

At run time, you can control how the model uses tools with `tool_choice`.

```python
result = agent.run(
    "What's the weather in Milan?",
    tool_choice="required_first",
)
```

Supported values:

- `"auto"`: the model decides whether to use a tool
- `"required"`: the model must use a tool every step
- `"none"`: the model must not use tools
- `"required_first"`: the first step must use a tool, later steps go back to `auto`
- `list[str]`: restrict tool use to a named subset

## Add memory to your agent

You can pass a custom `Memory` object.
This is useful when you want to start one specific run from custom history without changing the agent's default memory.

```python
from datapizza.memory import Memory
from datapizza.type import ROLE, TextBlock

memory = Memory()
memory.add_turn(TextBlock(content="The user's name is Federico."), role=ROLE.USER)

result = agent.run("What is the user's name?", memory=memory)
print(result.text)
```
## Stream responses

Use `stream_invoke(...)` when you want to observe the run as it happens.

It yields:

- `ClientResponse` chunks when client streaming is enabled
- `StepResult` objects for completed agent steps
- `Plan` objects when planning is enabled

```python
from datapizza.agents import Agent, StepResult
from datapizza.clients.openai import OpenAIClient
from datapizza.core.clients import ClientResponse

agent = Agent(
    name="assistant",
    system_prompt="You are a helpful assistant.",
    client=OpenAIClient(api_key="YOUR_API_KEY", model="gpt-4o-mini"),
    stream=True,
)

for event in agent.stream_invoke("Tell me a short joke."):
    if isinstance(event, ClientResponse):
        print(event.delta, end="", flush=True)
    elif isinstance(event, StepResult):
        print("\nfinal step:", event.text)
```

Async streaming works the same way with `a_stream_invoke(...)`.

```python
import asyncio


async def main():
    agent = Agent(
        name="assistant",
        system_prompt="You are a helpful assistant.",
        client=OpenAIClient(api_key="YOUR_API_KEY", model="gpt-4o-mini"),
        stream=True,
    )

    async for event in agent.a_stream_invoke("Tell me a short joke."):
        print(event)


asyncio.run(main())
```

## Return structured data

If you want typed output instead of plain text, set `output_cls`.

```python
from pydantic import BaseModel
from datapizza.agents import Agent
from datapizza.clients.openai import OpenAIClient


class Person(BaseModel):
    name: str
    age: int


agent = Agent(
    name="person_extractor",
    system_prompt="Extract a person from the input.",
    client=OpenAIClient(api_key="YOUR_API_KEY", model="gpt-4.1-mini"),
    output_cls=Person,
)

result = agent.run('{"name": "Alice", "age": 30}')
person = result.structured_data[0]
print(person.name)
```

When `output_cls` is set:

- the agent asks the client for structured output on each model turn
- the parsed objects are available in `result.structured_data`
- `result.text` may be empty

If the selected client does not support structured output, Datapizza raises a clear `ValueError`.

## Choose a multi-agent pattern

Before adding more agents, decide who should own the final answer.

- `can_call(...)` / `as_tool()`: one orchestrator stays in control and calls specialists as tools
- `handoffs`: control transfers to another agent, which continues the run


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datapizza-labs/datapizza-ai](https://github.com/datapizza-labs/datapizza-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
