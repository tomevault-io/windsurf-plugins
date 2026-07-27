---
trigger: always_on
description: Creates an agent, providing functionality identical to the official Langchain `create_agent`, but extends the model specification via string.
---

# Agent Module API Reference Documentation

## create_agent

Creates an agent, providing functionality identical to the official Langchain `create_agent`, but extends the model specification via string.

### Function Signature

```python
def create_agent(  # noqa: PLR0915
    model: str,
    tools: Sequence[BaseTool | Callable | dict[str, Any]] | None = None,
    *,
    system_prompt: str | SystemMessage | None = None,
    response_format: ResponseFormat[ResponseT] | type[ResponseT] | None = None,
    middleware: Sequence[AgentMiddleware[StateT_co, ContextT]] = (),
    state_schema: type[AgentState[ResponseT]] | None = None,
    context_schema: type[ContextT] | None = None,
    checkpointer: Checkpointer | None = None,
    store: BaseStore | None = None,
    interrupt_before: list[str] | None = None,
    interrupt_after: list[str] | None = None,
    debug: bool = False,
    name: str | None = None,
    cache: BaseCache | None = None,
) -> CompiledStateGraph[
    AgentState[ResponseT], ContextT, _InputAgentState, _OutputAgentState[ResponseT]
]:
```

### Parameters

| Parameter | Type | Required | Default | Description |
|------|------|------|--------|------|
| model | str | Yes | - | Model identifier string that can be loaded by `load_chat_model`. Can be specified in "provider:model-name" format |
| tools | Sequence[BaseTool \| Callable \| dict[str, Any]] \| None | No | None | List of tools available to the agent |
| system_prompt | str \| SystemMessage \| None | No | None | Custom system prompt for the agent |
| middleware | Sequence[AgentMiddleware[AgentState[ResponseT], ContextT]] | No | () | Middleware for the agent |
| response_format | ResponseFormat[ResponseT] \| type[ResponseT] \| None | No | None | Response format for the agent |
| state_schema | type[AgentState[ResponseT]] \| None | No | None | State schema for the agent |
| context_schema | type[ContextT] \| None | No | None | Context schema for the agent |
| checkpointer | Checkpointer \| None | No | None | Checkpointer for state persistence |
| store | BaseStore \| None | No | None | Store for data persistence |
| interrupt_before | list[str] \| None | No | None | Nodes to interrupt before execution |
| interrupt_after | list[str] \| None | No | None | Nodes to interrupt after execution |
| debug | bool | No | False | Enable debug mode |
| name | str \| None | No | None | Agent name |
| cache | BaseCache \| None | No | None | Cache |


### Notes

This function provides functionality identical to the official `langchain` `create_agent`, but extends model selection. The main difference is that the `model` parameter must be a string loadable by the `load_chat_model` function, allowing for more flexible model selection using registered model providers.

### Example

```python
agent = create_agent(model="vllm:qwen2.5-7b", tools=[get_current_time])
```

---

## wrap_agent_as_tool

Wraps an agent as a tool.

### Function Signature

```python
def wrap_agent_as_tool(
    agent: CompiledStateGraph,
    tool_name: Optional[str] = None,
    tool_description: Optional[str] = None,
    pre_input_hooks: Optional[
        tuple[
            Callable[[str, ToolRuntime], str | dict[str, Any]],
            Callable[[str, ToolRuntime], Awaitable[str | dict[str, Any]]],
        ]
        | Callable[[str, ToolRuntime], str | dict[str, Any]]
    ] = None,
    post_output_hooks: Optional[
        tuple[
            Callable[[str, dict[str, Any], ToolRuntime], Any],
            Callable[[str, dict[str, Any], ToolRuntime], Awaitable[Any]],
        ]
        | Callable[[str, dict[str, Any], ToolRuntime], Any]
    ] = None,
) -> BaseTool:
```

### Parameters

| Parameter | Type | Required | Default | Description |
|------|------|------|--------|------|
| agent | CompiledStateGraph | Yes | - | The agent |
| tool_name | Optional[str] | No | None | Tool name |
| tool_description | Optional[str] | No | None | Tool description |
| pre_input_hooks | - | No | None | Agent input preprocessing function |
| post_output_hooks | - | No | None | Agent output post-processing function |


### Example

```python
tool = wrap_agent_as_tool(agent)
```

---

## wrap_all_agents_as_tool

Wraps all agents as a single tool.

### Function Signature

```python
def wrap_all_agents_as_tool(
    agents: list[CompiledStateGraph],
    tool_name: Optional[str] = None,
    tool_description: Optional[str] = None,
    pre_input_hooks: Optional[
        tuple[
            Callable[[str, ToolRuntime], str | dict[str, Any]],
            Callable[[str, ToolRuntime], Awaitable[str | dict[str, Any]]],
        ]
        | Callable[[str, ToolRuntime], str | dict[str, Any]]
    ] = None,
    post_output_hooks: Optional[
        tuple[
            Callable[[str, dict[str, Any], ToolRuntime], Any],
            Callable[[str, dict[str, Any], ToolRuntime], Awaitable[Any]],
        ]
        | Callable[[str, dict[str, Any], ToolRuntime], Any]
    ] = None,
) -> BaseTool:
```


### Parameters

| Parameter | Type | Required | Default | Description |
|------|------|------|--------|------|
| agents | list[CompiledStateGraph] | Yes | - | List of agents (must contain at least 2, and each agent must have a unique name) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TBice123123/langchain-dev-utils](https://github.com/TBice123123/langchain-dev-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
