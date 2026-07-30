---
trigger: always_on
description: The Agentic Logic primitive provides **pluggable agents** that handle queries with varying levels of sophistication -- from simple single-turn responses to multi-turn tool-calling loops, ReAct-style reasoning, CodeAct code execution, recursive decomposition, and external agent communication.
---

# Agentic Logic Primitive

The Agentic Logic primitive provides **pluggable agents** that handle queries with varying levels of sophistication -- from simple single-turn responses to multi-turn tool-calling loops, ReAct-style reasoning, CodeAct code execution, recursive decomposition, and external agent communication.

---

## BaseAgent ABC

All agents implement the `BaseAgent` abstract base class, which provides both the `run()` contract and concrete helper methods that eliminate boilerplate in subclasses:

```python
class BaseAgent(ABC):
    agent_id: str
    accepts_tools: bool = False  # overridden by ToolUsingAgent

    def __init__(
        self,
        engine: InferenceEngine,
        model: str,
        *,
        bus: Optional[EventBus] = None,
        temperature: float = 0.7,
        max_tokens: int = 1024,
    ) -> None: ...

    @abstractmethod
    def run(
        self,
        input: str,
        context: Optional[AgentContext] = None,
        **kwargs: Any,
    ) -> AgentResult:
        """Execute the agent on *input* and return an AgentResult."""
```

### Class Attribute: `accepts_tools`

The `accepts_tools` class attribute (default `False`) enables the CLI and SDK to auto-detect which agents support tool-passing. Agents that set `accepts_tools = True` can receive `--tools` on the CLI and `tools=` in the SDK.

### Concrete Helper Methods

`BaseAgent` provides five concrete helpers that subclasses use to avoid duplicating common logic:

| Helper | Purpose |
|--------|---------|
| `_emit_turn_start(input)` | Publish `AGENT_TURN_START` on the event bus |
| `_emit_turn_end(**data)` | Publish `AGENT_TURN_END` on the event bus |
| `_build_messages(input, context, *, system_prompt)` | Assemble the message list from optional system prompt, conversation context, and user input |
| `_generate(messages, **extra_kwargs)` | Call `engine.generate()` with stored defaults (model, temperature, max_tokens) |
| `_max_turns_result(tool_results, turns, content)` | Build the standard `AgentResult` for when `max_turns` is exceeded |
| `_strip_think_tags(text)` | Remove `<think>...</think>` blocks from model output (static method) |

### The `run()` Contract

The `run()` method is the single entry point for all agent implementations. It receives:

- **`input`** -- The user's query text
- **`context`** -- An optional `AgentContext` with conversation history, tool names, and memory results
- **`**kwargs`** -- Additional implementation-specific parameters

It returns an `AgentResult` containing the response content, any tool results, the number of turns taken, and metadata.

### Supporting Dataclasses

```python
@dataclass(slots=True)
class AgentContext:
    conversation: Conversation    # Prior messages for multi-turn context
    tools: List[str]              # Available tool names
    memory_results: List[Any]     # Pre-fetched memory search results
    metadata: Dict[str, Any]      # Arbitrary key-value pairs

@dataclass(slots=True)
class AgentResult:
    content: str                  # The agent's response text
    tool_results: List[ToolResult]  # Results from tool invocations
    turns: int                    # Number of inference turns taken
    metadata: Dict[str, Any]      # Arbitrary metadata
```

---

## ToolUsingAgent

`ToolUsingAgent` is an intermediate base class for agents that accept and use tools. It extends `BaseAgent` with:

- **`accepts_tools = True`** -- Enables CLI/SDK tool introspection
- **`ToolExecutor`** -- Initialized from the provided tool list, handles dispatch with JSON argument parsing, latency tracking, and event bus integration
- **`max_turns`** -- Configurable loop iteration limit (default: 10)

```python
class ToolUsingAgent(BaseAgent):
    accepts_tools: bool = True

    def __init__(
        self,
        engine: InferenceEngine,
        model: str,
        *,
        tools: Optional[List[BaseTool]] = None,
        bus: Optional[EventBus] = None,
        max_turns: int = 10,
        temperature: float = 0.7,
        max_tokens: int = 1024,
    ) -> None: ...
```

All tool-using agents (`OrchestratorAgent`, `NativeReActAgent`, `NativeOpenHandsAgent`, `RLMAgent`) extend this class.

!!! info "Agents that bypass ToolUsingAgent"
    Some agents extend `BaseAgent` directly and set `accepts_tools = False`: `SimpleAgent` (single-turn, no tools), `OpenHandsAgent` (tool management is handled by the openhands-sdk), and `ClaudeCodeAgent` (tools are managed by the Claude Agent SDK). `SandboxedAgent` also extends `BaseAgent` directly because it wraps another agent rather than calling tools itself.

---

## Choosing an Agent

Start here. Pick the simplest agent that handles your task — simpler agents are faster, use fewer tokens, and are easier to debug. Reach for more complex agents only when the task demands it.

| Use case | Agent | Why |
|---|---|---|
| Simple Q&A, single-turn | `simple` | No overhead, one inference call |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-jarvis/OpenJarvis](https://github.com/open-jarvis/OpenJarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
