---
trigger: always_on
description: The agent component enables building autonomous AI agents that use workflows as tools. It implements a ReAct (Reasoning + Acting) loop where an LLM iteratively reasons about a task, calls tools, and processes results until a final answer is produced.
---

# Agent Component

The agent component enables building autonomous AI agents that use workflows as tools. It implements a ReAct (Reasoning + Acting) loop where an LLM iteratively reasons about a task, calls tools, and processes results until a final answer is produced.

## Basic Configuration

```yaml
component:
  type: agent
  tools:
    - search-web
    - analyze-image
  action:
    model:
      component: gpt-4o
      input:
        messages: ${messages}
        tools: ${tools}
    system_prompt: You are a helpful research assistant.
    user_prompt: ${input.question}
```

## Configuration Options

### Component Settings

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `type` | string | **required** | Must be `agent` |
| `tools` | array | `[]` | List of workflow IDs to use as tools |
| `max_iteration_count` | integer | `10` | Maximum number of ReAct loop iterations |
| `actions` | array | `[]` | List of agent actions |

### Action Configuration

Agent actions support the following options:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `model` | object | **required** | LLM model configuration for this action |
| `system_prompt` | any | `null` | System prompt. When set, a system message is prepended to the conversation |
| `user_prompt` | any | `null` | User prompt. Supports variable interpolation |
| `max_iteration_count` | integer | `null` | Maximum iterations (overrides component-level setting) |

If `user_prompt` is not specified, the agent's component input is used as the user message.

### Model Configuration

The `model` object specifies which component to use for LLM calls:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `component` | string | **required** | ID of the component to use for LLM calls |
| `action` | string | `__default__` | ID of the action to invoke on the component |
| `input` | object | `{}` | Input mapping from agent internal state to component input |

### Model Input Variables

The `model.input` mapping supports the following variables:

| Variable | Type | Description |
|----------|------|-------------|
| `${messages}` | array | Conversation message history (user, assistant, tool messages) managed by the agent |
| `${tools}` | array | Function calling schemas automatically generated from workflow tool definitions |
| `${input.*}` | any | Agent's original input variables (e.g., `${input.question}`) |

## Usage Examples

### Simple Agent with Tools

```yaml
components:
  - id: gpt-4o
    type: http-client
    base_url: https://api.openai.com/v1
    headers:
      Authorization: Bearer ${env.OPENAI_API_KEY}
    action:
      path: /chat/completions
      body:
        model: gpt-4o
        messages: ${input.messages}
        tools: ${input.tools}
      output: ${response.choices[0].message}

  - id: research-agent
    type: agent
    tools:
      - search-web
    action:
      model:
        component: gpt-4o
        input:
          messages: ${messages}
          tools: ${tools}
      system_prompt: You are a helpful research assistant.
      user_prompt: ${input.question}

workflows:
  - id: search-web
    description: "Search the web for information"
    jobs:
      - component: tavily
        input:
          query: ${input.query}
```

### Multi-Action Agent

```yaml
component:
  type: agent
  tools:
    - search-web
    - fetch-url
    - analyze-image
  max_iteration_count: 15
  actions:
    - id: research
      model:
        component: gpt-4o
        input:
          messages: ${messages}
          tools: ${tools}
      system_prompt: You are a thorough research assistant.
      user_prompt: ${input.question}

    - id: summarize
      model:
        component: claude-sonnet
        input:
          messages: ${messages}
      system_prompt: You are a concise summarizer.
      user_prompt: ${input.text}
      max_iteration_count: 1
```

### Agent with Custom Model Action

```yaml
components:
  - id: llm-server
    type: http-client
    base_url: http://localhost:8000/v1
    actions:
      - id: chat
        path: /chat/completions
        body:
          model: llama-3.1-70b
          messages: ${input.messages}
          tools: ${input.tools}
          temperature: 0.7
        output: ${response.choices[0].message}

      - id: chat-precise
        path: /chat/completions
        body:
          model: llama-3.1-70b
          messages: ${input.messages}
          tools: ${input.tools}
          temperature: 0.1
        output: ${response.choices[0].message}

  - id: coding-agent
    type: agent
    tools:
      - read-file
      - write-file
      - run-tests
    action:
      model:
        component: llm-server
        action: chat-precise
        input:
          messages: ${messages}
          tools: ${tools}
      system_prompt: You are an expert coding assistant. Read files, write code, and run tests to complete the task.
      user_prompt: ${input.task}
```

### Agent without Tools

An agent can operate without tools for single-turn inference:

```yaml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hanyeol/model-compose](https://github.com/hanyeol/model-compose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
