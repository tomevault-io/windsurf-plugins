---
trigger: always_on
description: This document explains how to use the Agent component of the Agent SDK.
---

# Agent

This document explains how to use the Agent component of the Agent SDK.

## Overview

The Agent is the core component of the SDK that coordinates the LLM, memory, and tools to create an intelligent assistant that can understand and respond to user queries.

## Creating an Agent

There are two main ways to create an agent: using Go code with options or loading from a YAML configuration file.

### Method 1: Using Go Code with Options

To create a new agent programmatically, use the `NewAgent` function with various options:

```go
import (
    "github.com/Ingenimax/agent-sdk-go/pkg/agent"
    "github.com/Ingenimax/agent-sdk-go/pkg/llm/openai"
    "github.com/Ingenimax/agent-sdk-go/pkg/memory"
)

// Create a new agent
agent, err := agent.NewAgent(
    agent.WithLLM(openaiClient),
    agent.WithMemory(memory.NewConversationBuffer()),
    agent.WithSystemPrompt("You are a helpful AI assistant."),
)
if err != nil {
    log.Fatalf("Failed to create agent: %v", err)
}
```

### Method 2: Using YAML Configuration

You can load agent configurations from YAML files using `LoadAgentConfigsFromFile` and create agents with `NewAgentFromConfig`:

```go
import (
    "github.com/Ingenimax/agent-sdk-go/pkg/agent"
    "github.com/Ingenimax/agent-sdk-go/pkg/llm/openai"
)

// Load agent configurations from YAML file
configs, err := agent.LoadAgentConfigsFromFile("agents.yaml")
if err != nil {
    log.Fatalf("Failed to load agent configs: %v", err)
}

// Create LLM client
llm := openai.NewClient(os.Getenv("OPENAI_API_KEY"))

// Create agent from configuration
agentInstance, err := agent.NewAgentFromConfig("file_analyzer", configs, nil, agent.WithLLM(llm))
if err != nil {
    log.Fatalf("Failed to create agent from config: %v", err)
}
```

## Agent Options

The Agent can be configured with various options:

### WithLLM

Sets the LLM provider for the agent:

```go
agent.WithLLM(openaiClient)
```

### WithMemory

Sets the memory system for the agent:

```go
agent.WithMemory(memory.NewConversationBuffer())
```

### WithTools

Adds tools to the agent:

```go
agent.WithTools(
    websearch.New(googleAPIKey, googleSearchEngineID),
    calculator.New(),
)
```

### WithSystemPrompt

Sets the system prompt for the agent:

```go
agent.WithSystemPrompt("You are a helpful AI assistant specialized in answering questions about science.")
```

### WithOrgID

Sets the organization ID for multi-tenancy:

```go
agent.WithOrgID("org-123")
```

### WithTracer

Sets the tracer for observability:

```go
agent.WithTracer(langfuse.New(langfuseSecretKey, langfusePublicKey))
```

### WithGuardrails

Sets the guardrails for safety:

```go
agent.WithGuardrails(guardrails.New(guardrailsConfigPath))
```

## YAML Configuration

The YAML configuration system provides a powerful way to define agent configurations declaratively. Here's the complete structure and capabilities:

### Basic Agent Configuration

```yaml
# Example agent configuration
my_agent:
  role: "Data Analysis Expert"
  goal: "Analyze data and provide insights"
  backstory: "Expert in data analysis with years of experience"

  # Behavioral settings
  max_iterations: 10
  require_plan_approval: false

  # LLM configuration
  llm_config:
    temperature: 0.3
    top_p: 0.9
    enable_reasoning: true
    reasoning_budget: 20000

  # Stream configuration
  stream_config:
    buffer_size: 100
    include_tool_progress: true
    include_intermediate_messages: false

  # Runtime settings
  runtime:
    log_level: "info"
    enable_tracing: true
    enable_metrics: true
    timeout: "30m"
```

### MCP Server Configuration

Configure Model Context Protocol (MCP) servers for extended capabilities:

```yaml
my_agent:
  mcp:
    mcpServers:
      filesystem:
        command: "npx"
        args: ["-y", "@modelcontextprotocol/server-filesystem", "."]

      database:
        command: "python"
        args: ["-m", "mcp_server_database"]
        env:
          DATABASE_URL: "${DATABASE_URL}"
```

### Tool Configuration

Configure various types of tools for your agent:

```yaml
my_agent:
  tools:
    # Built-in tools
    - type: "builtin"
      name: "calculator"
      enabled: true

    - type: "builtin"
      name: "websearch"
      enabled: true
      config:
        api_key: "${GOOGLE_API_KEY}"
        search_engine_id: "${GOOGLE_SEARCH_ENGINE_ID}"

    # Custom tools
    - type: "custom"
      name: "custom_analyzer"
      description: "Custom analysis tool"
      config:
        endpoint: "https://api.example.com/analyze"

    # Agent tools (calling other agents)
    - type: "agent"
      name: "specialist_agent"
      url: "http://specialist-service:8080"
      timeout: "5m"
```

### Memory Configuration

Configure different memory backends:

```yaml
my_agent:
  memory:
    # Buffer memory (default)
    type: "buffer"
    config:
      max_tokens: 4000

  # OR Redis memory
  # memory:
  #   type: "redis"
  #   config:
  #     address: "localhost:6379"
  #     db: 0

  # OR Vector memory
  # memory:
  #   type: "vector"
  #   config:
  #     provider: "weaviate"
  #     endpoint: "http://localhost:8080"
```

### Sub-Agents Configuration

Create hierarchical agent structures with sub-agents:

```yaml
main_agent:
  role: "Project Coordinator"
  goal: "Coordinate complex projects"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ingenimax/agent-sdk-go](https://github.com/Ingenimax/agent-sdk-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
