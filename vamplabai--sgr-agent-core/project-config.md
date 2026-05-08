---
trigger: always_on
description: Rules for core modules of SGR Agent Core
---


# Rules for Core Modules

## Base Classes

### BaseAgent (`sgr_agent_core/base_agent.py`)
- Parent class for all agents
- Implements three-phase execution cycle: Reasoning → Select Action → Action
- Manages agent context, conversation history, and streaming
- Must be subclassed to implement `_reasoning_phase()`, `_select_action_phase()`, `_action_phase()`
- Automatically registered in `AgentRegistry` via `AgentRegistryMixin`
- Key attributes:
  - `id`: Unique agent identifier (format: `{def_name or name}_{uuid}`)
  - `name`: Agent class name
  - `task_messages`: Initial task messages in OpenAI format
  - `config`: AgentConfig instance with all settings
  - `openai_client`: AsyncOpenAI client for LLM API
  - `toolkit`: List of tool classes available to agent
  - `_context`: AgentContext instance with execution state
  - `conversation`: List of messages in OpenAI format for LLM context
  - `streaming_generator`: OpenAIStreamingGenerator for streaming responses
  - `logger`: Logger instance for agent logging
  - `log`: List of execution logs
  - `creation_time`: Datetime when agent was created
- Key methods:
  - `execute()`: Main execution loop (called externally)
  - `_execution_step()`: Single step of execution cycle (can be overridden)
  - `_prepare_context()`: Prepare conversation context (can be overridden)
  - `_prepare_tools()`: Prepare available tools (can be overridden)
  - `provide_clarification()`: Receive clarification from external source
  - `_log_reasoning()`: Log reasoning phase results
  - `_log_tool_execution()`: Log tool execution results
  - `_save_agent_log()`: Save execution log to file

### BaseTool (`sgr_agent_core/base_tool.py`)
- Parent class for all tools (Pydantic `BaseModel`)
- Class variables: `tool_name` (ClassVar[str]), `description` (ClassVar[str])
- Must implement `__call__(context: AgentContext, config: AgentConfig, **kwargs) -> str`
- Returns string or JSON string from `__call__()`
- Automatically registered in `ToolRegistry` via `ToolRegistryMixin`
- `tool_name` defaults to class name (lowercase) if not set
- `description` defaults to class docstring if not set

### MCPBaseTool (`sgr_agent_core/base_tool.py`)
- Base class for MCP-integrated tools (inherits from `BaseTool`)
- Class variable: `_client` (ClassVar[Client | None]) - MCP client instance
- `__call__()`: Calls MCP tool via `fastmcp.Client.call_tool()`
- Converts MCP responses to JSON string
- Respects `mcp_context_limit` from `ExecutionConfig`
- Handles errors gracefully (returns error message as string)

## Configuration Modules

### GlobalConfig (`sgr_agent_core/agent_config.py`)
- Singleton pattern for global configuration
- All calls to `GlobalConfig()` return the same instance
- Loads from YAML files via `from_yaml()` method
- Loads from environment variables via `pydantic-settings` (prefix `SGR__`)
- Contains: `llm`, `execution`, `prompts`, `mcp`, `agents`, `tools`
- `agents`: Dictionary of `AgentDefinition` instances by name
- `tools`: Dictionary of tool definitions by name
- `definitions_from_yaml()`: Loads agent definitions from YAML (merges with existing)

### AgentDefinition (`sgr_agent_core/agent_definition.py`)
- Definition template for creating agents
- Inherits from `AgentConfig` (has all config fields)
- Additional fields: `name`, `base_class`, `tools`
- `base_class`: Can be class, ImportString (e.g., `"sgr_agent_core.agents.SGRAgent"`), or registry name
- `tools`: List of `ToolDefinition` objects (resolved from names, classes, or dicts)
- Supports YAML loading via `GlobalConfig.definitions_from_yaml()`
- Validates import strings point to existing files
- Automatically merges with `GlobalConfig` defaults (tools: global kwargs merged with agent-level kwargs)

### AgentConfig (`sgr_agent_core/agent_definition.py`)
- Runtime configuration for agent instance
- Combines: `LLMConfig`, `ExecutionConfig`, `PromptsConfig`, `MCPConfig`
- Note: Search settings (`tavily_api_key`, `max_results`, etc.) are NOT in `AgentConfig`; they are configured per-tool in the `tools:` section as `SearchConfig` kwargs
- Supports hierarchical inheritance from `GlobalConfig`
- Uses `extra="allow"` to support custom fields for agent-specific parameters

## Factory and Services

### AgentFactory (`sgr_agent_core/agent_factory.py`)
- Creates agent instances from `AgentDefinition`
- Resolves agent classes from `AgentRegistry` (by name or ImportString)
- Resolves tools from `ToolRegistry` or `config.tools` section
- Tool resolution order:
  1. Tools defined in `config.tools` section
  2. Tools in `ToolRegistry` by name (snake_case or PascalCase)
  3. Auto-conversion snake_case → PascalCase for backward compatibility
- Builds MCP tools via `MCP2ToolConverter`
- Creates OpenAI client with proxy support via `httpx.AsyncClient`
- `get_definitions_list()`: Returns all agent definitions from `GlobalConfig`

### AgentRegistry (`sgr_agent_core/services/registry.py`)
- Registry for agent classes (subclass of `Registry[BaseAgent]`)
- Automatic registration via `AgentRegistryMixin` in `BaseAgent.__init_subclass__()`
- Registers by class name (lowercase) and `name` attribute
- Supports lookup by name (case-insensitive)

### ToolRegistry (`sgr_agent_core/services/registry.py`)
- Registry for tool classes (subclass of `Registry[BaseTool]`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vamplabAI/sgr-agent-core](https://github.com/vamplabAI/sgr-agent-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
