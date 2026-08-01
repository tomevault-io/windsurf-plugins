---
trigger: always_on
description: This guide covers how to define and use custom agents (personas) in the Claude Agent SDK for Elixir.
---

# Agents Guide

This guide covers how to define and use custom agents (personas) in the Claude Agent SDK for Elixir.

## Table of Contents

1. [What are Agents](#what-are-agents)
2. [Agent.new/1 and the Agent Struct](#agentnew1-and-the-agent-struct)
3. [Agent Configuration](#agent-configuration)
4. [Using Agents in Options](#using-agents-in-options)
5. [Switching Agents at Runtime](#switching-agents-at-runtime)
6. [Multi-Agent Workflows](#multi-agent-workflows)
7. [Filesystem Agents](#filesystem-agents)
8. [Agent Validation](#agent-validation)
9. [Best Practices](#best-practices)

---

## What are Agents

Agents are custom personas or roles that you can define for Claude. Each agent has its own:

- **Description**: A human-readable description of what the agent does
- **Prompt**: A system prompt that defines the agent's behavior and expertise
- **Allowed Tools**: An optional list of tools the agent can use
- **Disallowed Tools**: An optional list of tools the agent cannot use
- **Model**: An optional model specification (e.g., "haiku", "sonnet", "opus")
- **Skills**: An optional list of skill names the agent has access to
- **MCP Servers**: An optional list of MCP server names or configurations
- **Max Turns**: An optional turn limit for the agent

Agents enable you to:

- Create specialized AI assistants for different tasks
- Switch between different Claude behaviors at runtime
- Maintain conversation context across agent switches
- Build multi-agent workflows where different agents handle different parts of a task

### Use Cases

- **Code Review Agent**: Analyzes code for bugs, security issues, and best practices
- **Documentation Agent**: Writes clear, comprehensive documentation
- **Testing Agent**: Creates test cases and validates implementations
- **Research Agent**: Gathers information and provides analysis
- **Refactoring Agent**: Improves code structure and performance

---

## Agent.new/1 and the Agent Struct

The `ClaudeAgentSDK.Agent` module provides the `Agent` struct and functions for creating and managing agents.

### The Agent Struct

```elixir
%ClaudeAgentSDK.Agent{
  name: atom() | nil,                      # Optional identifier
  description: String.t(),                  # Required: What the agent does
  prompt: String.t(),                       # Required: System prompt
  allowed_tools: [String.t()] | nil,        # Optional: List of allowed tool names
  disallowed_tools: [String.t()] | nil,     # Optional: List of disallowed tool names
  model: String.t() | nil,                  # Optional: Model to use
  skills: [String.t()] | nil,               # Optional: Skill names
  mcp_servers: [String.t() | map()] | nil,  # Optional: MCP server names/configs
  max_turns: pos_integer() | nil            # Optional: Turn limit
}
```

### Creating Agents with Agent.new/1

The `ClaudeAgentSDK.Agent.new/1` function creates a new agent struct from a keyword list:

```elixir
alias ClaudeAgentSDK.Agent

# Minimal agent (only required fields)
simple_agent = Agent.new(
  description: "A helpful assistant",
  prompt: "You are a helpful assistant that provides clear, concise answers."
)

# Complete agent with all fields
code_reviewer = Agent.new(
  name: :code_reviewer,
  description: "Expert code reviewer",
  prompt: """
  You are an expert code reviewer. When reviewing code:
  - Check for bugs and logic errors
  - Identify security vulnerabilities
  - Suggest performance improvements
  - Enforce coding standards and best practices
  Provide concise, actionable feedback.
  """,
  allowed_tools: ["Read", "Grep", "Glob"],
  model: "sonnet"
)
```

### Required Fields

- `:description` - A non-empty string describing the agent's purpose
- `:prompt` - A non-empty string defining the agent's behavior

### Optional Fields

- `:name` - An atom identifier for the agent (useful for referencing in multi-agent setups)
- `:allowed_tools` - A list of tool name strings the agent can use
- `:disallowed_tools` - A list of tool name strings the agent cannot use
- `:model` - A string specifying which model to use (e.g., `"haiku"`, `"sonnet"`, `"opus"`) -- see [Model Configuration](model-configuration.md)
- `:skills` - A list of skill name strings (e.g., `["code-review", "testing"]`)
- `:mcp_servers` - A list of MCP server names or configuration maps
- `:max_turns` - Maximum number of conversation turns for this agent

Tip: MCP tool names are always strings (`mcp__<server>__<tool>`). Avoid atom tool names in agent configs to prevent atom leaks.

---

## Agent Configuration

### Description

The description should clearly explain what the agent does. It helps users understand the agent's purpose and is used by the CLI for agent discovery.

```elixir
# Good descriptions
description: "Python coding expert that writes clean, type-hinted code"
description: "Security analyst that identifies vulnerabilities in code"
description: "Technical writer that creates clear API documentation"

# Avoid vague descriptions
description: "A helper"  # Too vague
description: "Agent"     # Not descriptive
```

### Prompt

The prompt is the system instruction that shapes the agent's behavior. Write detailed prompts that:

- Define the agent's expertise and role

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nshkrdotcom/claude_agent_sdk](https://github.com/nshkrdotcom/claude_agent_sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
