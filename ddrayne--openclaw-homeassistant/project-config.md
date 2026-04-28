---
trigger: always_on
description: This guide explains how OpenClaw agents work and how to configure them for optimal Home Assistant integration.
---

# Understanding OpenClaw Agents

This guide explains how OpenClaw agents work and how to configure them for optimal Home Assistant integration.

## Table of Contents

- [What is an Agent?](#what-is-an-agent)
- [Agent Architecture](#agent-architecture)
- [Agent Configuration](#agent-configuration)
- [Session Management](#session-management)
- [Multi-Agent Setup](#multi-agent-setup)
- [Voice-Optimized Agents](#voice-optimized-agents)
- [Agent Customization](#agent-customization)
- [Best Practices](#best-practices)

## What is an Agent?

A **OpenClaw agent** is an AI assistant instance with:
- A specific AI model (Claude, GPT, or local models)
- Custom system prompts and behavior
- Access to configured skills and integrations
- Memory and conversation context
- Unique identity and capabilities

Think of agents as different "personalities" or "experts" you can talk to, each configured for specific tasks.

## Agent Architecture

### Components

```
┌─────────────────────────────────────────────┐
│            OpenClaw System                  │
├─────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐    │
│  │ Agent   │  │ Agent   │  │ Agent   │    │
│  │ "main"  │  │ "codex" │  │ "voice" │    │
│  │         │  │         │  │         │    │
│  │ Model:  │  │ Model:  │  │ Model:  │    │
│  │ Sonnet  │  │ Opus    │  │ Sonnet  │    │
│  │         │  │         │  │         │    │
│  │ Skills: │  │ Skills: │  │ Skills: │    │
│  │ All     │  │ Coding  │  │ Basic   │    │
│  └─────────┘  └─────────┘  └─────────┘    │
├─────────────────────────────────────────────┤
│           Gateway (Port 18789)              │
└─────────────────────────────────────────────┘
         │                │               │
         ▼                ▼               ▼
    Telegram         Discord      Home Assistant
```

### Key Concepts

**Agent ID:** Unique identifier (e.g., `main`, `codex`, `voice-assistant`)

**Session:** A conversation thread with an agent. Multiple sessions can exist for the same agent.

**Session Key:** Unique identifier for a conversation session (e.g., `main`, `home-assistant-abc123`)

**Model:** The AI model powering the agent (e.g., `anthropic/claude-sonnet-4-5`)

**System Prompt:** Instructions that define the agent's behavior and personality

## Agent Configuration

### Default Agent Configuration

The default `main` agent is configured in `~/.openclaw/openclaw.json`:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-5"
      },
      "workspace": "/Users/dan/.openclaw/workspace",
      "heartbeat": {
        "every": "2h",
        "model": "sonnet"
      }
    },
    "list": [
      {
        "id": "main",
        "subagents": {
          "allowAgents": ["main", "codex"]
        }
      }
    ]
  }
}
```

### Creating Additional Agents

Add new agents to the `agents.list` array:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "model": "anthropic/claude-sonnet-4-5"
      },
      {
        "id": "voice-assistant",
        "name": "Voice Assistant",
        "model": "anthropic/claude-sonnet-4-5",
        "systemPrompt": "You are a voice assistant for a smart home..."
      },
      {
        "id": "codex",
        "name": "Codex",
        "model": "anthropic/claude-opus-4-5"
      }
    ]
  }
}
```

### Agent Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | Unique agent identifier (required) |
| `name` | string | Human-readable name |
| `model` | string | AI model to use |
| `systemPrompt` | string | Custom system prompt |
| `workspace` | string | Agent's working directory |
| `maxConcurrent` | number | Max concurrent requests |
| `subagents.allowAgents` | array | Which agents can spawn sub-agents |

## Session Management

### What is a Session?

A **session** represents a conversation thread with an agent. Each session maintains:
- Conversation history
- Context and memory
- Token usage statistics
- Last active timestamp

### Session Types

**Direct Sessions:**
- Created when you message an agent directly
- One session per communication channel (Telegram, Discord, etc.)
- Session key matches agent ID (e.g., `main`)

**Custom Sessions:**
- Created for specific purposes
- Can be isolated from other conversations
- Custom session keys (e.g., `home-assistant`, `automation-xyz`)

**Spawned Sessions:**
- Background task sessions
- Created by `sessions_spawn`
- Temporary (can be deleted after task completion)

### Configuring Home Assistant Session

In the integration configuration, you specify which session to use:

**Default (main session):**
```yaml
session_key: "main"
```
- HA conversations appear in your main OpenClaw session
- Shares context with other platforms (Telegram, Discord)
- Full conversation history available

**Dedicated session:**
```yaml
session_key: "home-assistant"
```
- Isolated conversation thread
- Separate from other OpenClaw interactions
- Clean separation of concerns

**Per-family-member:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddrayne/openclaw-homeassistant](https://github.com/ddrayne/openclaw-homeassistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
