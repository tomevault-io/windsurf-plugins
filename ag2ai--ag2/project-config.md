---
trigger: always_on
description: Use `just test` as alias for `pytest` execution to run the tests.
---

# test/ Guidelines

Use `just test` as alias for `pytest` execution to run the tests.

## Testing Conventions

Always write public API-based tests. Do not assert implementation details.

```python
# === BAD - digging into implementation details ===
async def test_collects_events_in_window(self) -> None:
    stream = MemoryStream()
    ctx = Context(stream=stream)
    batches: list = []

    async def callback(events, _ctx):
        batches.append(events)

    watch = CadenceWatch(max_wait=0.1, condition=ToolCallEvent)
    watch.arm(stream, callback)

    await stream.send(ToolCallEvent(name="t1", arguments="{}"), ctx)
    await stream.send(ToolCallEvent(name="t2", arguments="{}"), ctx)

    await asyncio.sleep(0.2)

    assert len(batches) == 1
    assert len(batches[0]) == 2


# === GOOD - public-api based test ===
async def test_collects_events_in_window(self) -> None:
    # arrange stream
    stream = MemoryStream()
    batches: list[BaseEvent] = []

    async def callback(events: BaseEvent, ctx: Context) -> None:
        batches.append(events)

    watch = CadenceWatch(max_wait=0.01, condition=ToolCallEvent)
    watch.arm(stream, callback)

    # arrange agent
    tool_calls = [
        ToolCallEvent(name="t1", arguments="{}"),
        ToolCallEvent(name="t2", arguments="{}"),
    ]

    agent = Agent(
        "test-agent",
        config=testing.TestConfig(tool_calls, "Done"),
    )

    @agent.tool
    def t1(): ...
    @agent.tool
    def t2(): ...

    # act
    await agent.ask("Hello, world!", stream=stream)
    await asyncio.sleep(0.02)

    # assert
    assert batches == [
        IsList(*tool_calls, check_order=False),
    ]
```

- Always use `ag2.testing.TestConfig` to mock LLM responses in agent-based tests.
- Always use `ag2.testing.TrackingConfig` to validate messages the framework sends to the LLM (for example: tool results and user input).

### No monkeypatching internals

Do not use `monkeypatch.setattr`, `setattr` on an instance, or `unittest.mock.patch` to swap out a private function, method, or attribute (anything `_prefixed`). Patching internals pins the test to *how* the code works today, so it keeps passing even after the real behavior breaks.

The agent's public seam is `config=`. Script the LLM's turns with `ag2.testing.TestConfig` instead of reaching for the agent's private client — the same test, before and after:

```python
from ag2 import Agent
from ag2.events import ToolCallEvent
from ag2.testing import TestConfig


# BAD — patch the agent's private client to script the turn. The test breaks the
# moment that internal is renamed, and a green run proves nothing about the wiring.
async def test_agent_answers_with_tool(monkeypatch):
    agent = Agent("weather", tools=[get_weather])
    monkeypatch.setattr(agent, "_client", _scripted_client(...))  # private attribute
    ...


# GOOD — script the same turns through the public `config=` seam.
async def test_agent_answers_with_tool():
    agent = Agent(
        "weather",
        config=TestConfig(
            ToolCallEvent(name="get_weather", arguments='{"city": "Tokyo"}'),  # turn 1: call the tool
            "It's sunny in Tokyo.",  # turn 2: final reply
        ),
        tools=[get_weather],
    )

    reply = await agent.ask("Weather in Tokyo?")

    assert reply.body == "It's sunny in Tokyo."
```

Use `TrackingConfig` (which wraps a `TestConfig`) when you also need to assert what the framework *sent* the LLM — again, no patching required.

Failure paths follow the same principle: raise from a public collaborator, never a patched internal — an `Agent` double whose `ask` raises, a `TraceSource` whose `load` raises, or a `LinkEndpoint` whose `frames()` raises (registered with `hub.attach_endpoint(...)`). If a branch can *only* be reached by patching an internal, it isn't publicly observable: cover the observable contract instead of reaching in to hit the line.

### Assertion style

Avoid chained field-access assertions like `result[0]["tool_calls"][0]["function"]["arguments"] == {...}`. Instead, compare the whole object directly (`assert msg == {...}`) or use **dirty-equals** `IsPartialDict` when only some fields matter:

```python
# Bad
assert result[0]["role"] == "assistant"
assert result[0]["tool_calls"][0]["function"]["arguments"] == {}

# Good — full comparison
assert result[0] == {"role": "assistant", "tool_calls": [...]}

# Good — partial match with dirty-equals (always use dict syntax, not kwargs)
from dirty_equals import IsPartialDict

assert result[0] == IsPartialDict({"role": "assistant"})  # Good
assert result[0] == IsPartialDict(role="assistant")  # Bad — use dict syntax
```

### Imports

All imports must be at the top of the test file. Never place imports inside individual test functions until user asks for it.

### Function vs class-based tests

Use **plain functions** for standalone tests. Use **classes** to group multiple related tests that share a logical subject (e.g., `TestImageUrlInput`, `TestBinaryInput`). Do not wrap a single test method in a class — keep it a plain function instead.

If you need to apply markers to each test in class, apply them to the class itself.

```python
# Bad - markers are applied to each test individually
class TestAgent:
    @pytest.mark.asyncio

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ag2ai/ag2](https://github.com/ag2ai/ag2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
