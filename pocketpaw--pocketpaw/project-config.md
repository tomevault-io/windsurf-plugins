---
trigger: always_on
description: This file gives GitHub Copilot repo-specific guidance so its suggestions align with PocketPaw's architecture and conventions. Read [`CLAUDE.md`](../CLAUDE.md) for the full picture; this file focuses on the most frequently violated rules.
---

# GitHub Copilot Instructions for PocketPaw

This file gives GitHub Copilot repo-specific guidance so its suggestions align with PocketPaw's architecture and conventions. Read [`CLAUDE.md`](../CLAUDE.md) for the full picture; this file focuses on the most frequently violated rules.

---

## 1. Event-driven message bus

All communication **must** flow through the message bus (`src/pocketpaw/bus/`). The three canonical event types are defined in `bus/events.py`:

| Type | Direction | Purpose |
|---|---|---|
| `InboundMessage` | Channel → Bus | User input from any channel |
| `OutboundMessage` | Bus → Channel | Agent response (supports streaming via `is_stream_chunk` / `is_stream_end`) |
| `SystemEvent` | Internal | Tool activity, errors, thinking — consumed by the dashboard Activity panel |

**Rules:**
- Adapters publish `InboundMessage` via `self._publish_inbound()`. They **never** call agent code directly.
- Agents subscribe to `InboundMessage` and publish `OutboundMessage` / `SystemEvent`. They **never** call adapter methods.
- Do not create new event types without discussion. Extend the metadata dict on existing types first.

---

## 2. Async everywhere

- Every method that performs I/O (network, file, subprocess) **must** be `async def`.
- Use `asyncio.to_thread()` for unavoidable blocking calls (e.g. a sync-only third-party SDK).
- Tests use `pytest-asyncio` in **`asyncio_mode = "auto"`** — no `@pytest.mark.asyncio` decorator needed.
- Never call `asyncio.run()` inside library code. Reserve it for entry points only.

```python
# ✅ correct
async def fetch_data(url: str) -> str:
    async with httpx.AsyncClient() as client:
        resp = await client.get(url)
    return resp.text

# ❌ wrong — blocks the event loop
def fetch_data(url: str) -> str:
    return requests.get(url).text
```

---

## 3. Protocol-oriented interfaces

Use `Protocol` (not `ABC`) for all public interfaces. Implement the protocol; do **not** subclass the concrete implementation class.

Key protocols:

| Protocol | Location |
|---|---|
| `AgentBackend` | `agents/backend.py` |
| `ToolProtocol` | `tools/protocol.py` |
| `ChannelAdapter` / `BaseChannelAdapter` | `bus/adapters/base.py` |
| `MemoryStoreProtocol` | `memory/protocol.py` |

```python
# ✅ correct — implement the protocol
from pocketpaw.agents.backend import AgentBackend

class MyBackend:
    async def run(self, prompt: str, **kwargs) -> AsyncIterator[AgentEvent]:
        ...

# ❌ wrong — inheriting from the concrete class couples you to internals
from pocketpaw.agents.claude_sdk import ClaudeAgentBackend

class MyBackend(ClaudeAgentBackend):  # noqa: don't do this
    ...
```

---

## 4. Standardised `AgentEvent` stream

Every backend that implements `AgentBackend` must yield `AgentEvent` objects with these fields:

```python
@dataclass
class AgentEvent:
    type: str      # see valid values below
    content: str   # text payload (may be empty string)
    metadata: dict # arbitrary extra data
```

**Valid `type` values** (do not invent new ones without updating `backend.py`):

| Value | Meaning |
|---|---|
| `"text"` | Streamed text chunk from the model |
| `"tool_start"` | A tool invocation is beginning |
| `"tool_result"` | A tool returned its result |
| `"thinking"` | Model is reasoning (extended thinking) |
| `"error"` | Non-fatal error; stream continues |
| `"done"` | Final event — stream is complete |

**Rule: always yield `"done"` last.** The agent loop depends on this to close the stream.

```python
async def run(self, prompt, **kwargs):
    async for chunk in self._model.stream(prompt):
        yield AgentEvent(type="text", content=chunk, metadata={})
    yield AgentEvent(type="done", content="", metadata={})  # required
```

---

## 5. Lazy imports for optional dependencies

Optional SDK imports (discord, slack, anthropic, openai, google-genai, etc.) **must** live inside `__init__` or the first-use method — **never at module scope**.

```python
# ✅ correct — import inside __init__ or first-use method
class DiscordAdapter(BaseChannelAdapter):
    def __init__(self, ...):
        try:
            import discord  # lazy import
            self._discord = discord
        except ImportError as exc:
            raise ImportError("Install pocketpaw[discord]") from exc

# ❌ wrong — top-level import breaks installs that don't have the dep
import discord  # fails for users without pocketpaw[discord]
```

Register new backends in `_BACKEND_REGISTRY` inside `agents/registry.py`:

```python
_BACKEND_REGISTRY: dict[str, str] = {
    "my_new_backend": "pocketpaw.agents.my_module.MyBackend",
    # dotted path is imported lazily by AgentRouter._initialize_agent()
}
```

---

## 6. Where new code goes

### Adding a new agent backend

1. Create `src/pocketpaw/agents/<name>.py` implementing the `AgentBackend` protocol.
2. Import the optional SDK lazily inside `__init__` or `_initialize()`.
3. Register the backend string key in `_BACKEND_REGISTRY` in `agents/registry.py`.
4. Add the optional dependency as an extra in `pyproject.toml` (e.g. `[project.optional-dependencies] mybackend = ["my-sdk>=1.0"]`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pocketpaw/pocketpaw](https://github.com/pocketpaw/pocketpaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
