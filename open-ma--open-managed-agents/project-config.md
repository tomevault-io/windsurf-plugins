---
trigger: always_on
description: This document covers the core concepts, lifecycle, and configuration of agents in Open Managed Agents.
---

# Agents Guide

This document covers the core concepts, lifecycle, and configuration of agents in Open Managed Agents.

---

## Core Concepts

Open Managed Agents is built around a **meta-harness** architecture with four key abstractions:

### Agent

An **agent** is a configuration object that defines _what_ an AI assistant can do. It specifies the model, system prompt, available tools, skills, and optional connections to other agents or MCP servers.

Agents are versioned — every update creates a new version. Sessions bind to a specific agent version at creation time.

### Session

A **session** is a running conversation between a user and an agent. It owns an append-only **event log** stored in a Durable Object backed by SQLite. Sessions are the unit of state — agents themselves are stateless configurations.

Sessions can be streamed in real-time via SSE, resumed after crashes, and archived when complete.

### Environment

An **environment** defines the execution sandbox — what packages are installed, what networking is allowed, and what container image to use. Environments are reusable across sessions and agents.

### Vault

A **vault** is a secure credential store. Credentials in vaults are **never exposed to sandboxes** — they're injected via an outbound proxy that intercepts HTTP requests and adds authentication headers transparently.

---

## Agent Lifecycle

```
                    ┌──────────┐
                    │  Create   │  POST /v1/agents
                    └────┬─────┘
                         │
                    ┌────▼─────┐
              ┌────►│  Active   │◄────┐
              │     └────┬─────┘     │
              │          │           │
         ┌────┴───┐ ┌───▼────┐ ┌───┴─────┐
         │ Update  │ │ Archive│ │ Sessions│
         │ (new    │ │        │ │ use it  │
         │ version)│ └───┬────┘ └─────────┘
         └─────────┘     │
                    ┌────▼─────┐
                    │ Archived  │
                    └──────────┘
```

1. **Create** — `POST /v1/agents` with name, model, system prompt, and tools
2. **Use** — Create sessions referencing the agent by ID
3. **Update** — `PUT /v1/agents/:id` creates a new version; existing sessions keep their original version
4. **Archive** — `POST /v1/agents/:id/archive` soft-deletes the agent

---

## Agent Configuration

### Minimal Agent

```json
{
  "name": "Assistant",
  "model": "claude-sonnet-4-6",
  "system": "You are a helpful assistant.",
  "tools": [{ "type": "agent_toolset_20260401" }]
}
```

### Full Configuration

```json
{
  "name": "Full-Stack Developer",
  "description": "A coding agent with access to tools, skills, and external services.",
  "model": "claude-sonnet-4-6",
  "system": "You are an expert full-stack developer. Write clean, tested code.",
  "tools": [
    {
      "type": "agent_toolset_20260401",
      "default_config": { "enabled": true },
      "configs": [
        { "name": "web_search", "enabled": false }
      ]
    },
    {
      "type": "custom",
      "name": "deploy",
      "description": "Deploy the application to production",
      "input_schema": {
        "type": "object",
        "properties": {
          "environment": { "type": "string", "enum": ["staging", "production"] }
        },
        "required": ["environment"]
      }
    }
  ],
  "mcp_servers": [
    { "name": "github", "type": "url", "url": "https://mcp.github.com/sse" }
  ],
  "skills": [
    { "skill_id": "skill_xxx", "type": "prompt" }
  ],
  "callable_agents": [
    { "type": "agent", "id": "agent_yyy" }
  ],
  "model_card_id": "mc_xxx",
  "aux_model": "claude-haiku-4-5",
  "harness": "default",
  "metadata": {
    "team": "platform",
    "owner": "alice"
  }
}
```

### Configuration Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | Display name for the agent |
| `description` | string | No | Human-readable description |
| `model` | string or object | Yes | Model identifier (e.g. `"claude-sonnet-4-6"`) or `{ id, speed }` |
| `system` | string | Yes | System prompt — defines the agent's behavior and persona |
| `tools` | array | No | Tool configurations (toolsets, custom tools) |
| `mcp_servers` | array | No | External MCP server connections |
| `skills` | array | No | Skill references to mount into the sandbox |
| `callable_agents` | array | No | Other agents this agent can delegate to |
| `model_card_id` | string | No | Reference to a model card for custom provider config |
| `aux_model` | string or object | No | Auxiliary model used by tools for in-process LLM work (e.g. `web_fetch` page summarization). Same shape as `model`. When unset, tools that would benefit from summarization fall back to returning raw content. |
| `aux_model_card_id` | string | No | Companion to `aux_model` — explicit model card binding when needed |
| `harness` | string | No | Harness implementation to use (default: `"default"`) |
| `metadata` | object | No | Arbitrary key-value metadata |

---

## Tools

### Built-in Toolset

The `agent_toolset_20260401` provides 8 tools designed for general-purpose agent work:

| Tool | Description | Key Behaviors |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-ma/open-managed-agents](https://github.com/open-ma/open-managed-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
