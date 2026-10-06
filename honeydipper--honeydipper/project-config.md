---
trigger: always_on
description: > **Introduced in v4** — The agent system is a major v4 feature that enables Honeydipper to interact with AI models through persistent, stateful conversations. It is fully integrated with the Honeydipper workflow engine, allowing agents to call systems, workflows, sub-agents, and remote MCP tools.
---

# Agent System Guide

> **Introduced in v4** — The agent system is a major v4 feature that enables Honeydipper to interact with AI models through persistent, stateful conversations. It is fully integrated with the Honeydipper workflow engine, allowing agents to call systems, workflows, sub-agents, and remote MCP tools.

## Table of Contents

- [Overview](#overview)
- [Core Concepts](#core-concepts)
  - [AgentSession](#agentsession)
  - [ConvoState](#convostate)
  - [PersistentAgentStore](#persistentagentstore)
- [Configuration](#configuration)
  - [Agent Definition](#agent-definition)
  - [CompactionPolicy](#compactionpolicy)
  - [AgentToolDef](#agenttooldef)
- [Tool Building](#tool-building)
  - [System Tools (`sys_` prefix)](#system-tools-sys_-prefix)
  - [Workflow Tools (`wf__` prefix)](#workflow-tools-wf__-prefix)
  - [Agent Tools (`ag__` prefix)](#agent-tools-ag__-prefix)
  - [MCP Tools (`mcp__` prefix)](#mcp-tools-mcp__-prefix)
  - [Skill Loading Tool (`hd_load_skill`)](#skill-loading-tool-hd_load_skill)
- [Polling Mechanism](#polling-mechanism)
- [Compaction Policies](#compaction-policies)
- [Message Flow](#message-flow)
- [Pre-Context and Skills](#pre-context-and-skills)
- [Session Lifecycle](#session-lifecycle)
- [Practical Examples](#practical-examples)

---

## Overview

The agent system enables Honeydipper to run stateful, multi-turn conversations with AI models. Unlike traditional stateless API calls to LLMs, the agent system:

- **Persists conversation history** across sessions using a distributed cache (Redis), surviving daemon restarts and configuration reloads.
- **Supports tool calling** — agents can invoke Honeydipper systems, workflows, other agents, and remote MCP servers.
- **Emits streaming responses** back to the caller (workflow or UI) as the model generates output.
- **Auto-compacts** conversation history using configurable LLM-based summarization to stay within token limits.
- **Integrates seamlessly** with the Honeydipper eventbus — workflows can trigger agents and await their results.

---

## Core Concepts

### AgentSession

`AgentSession` (`internal/agent/agent_session.go`) holds the runtime state of a single agent inference or chat-turn session. It is the central object in the agent system.

| Field | Type | Description |
|---|---|---|
| `ID` | `string` | Unique session identifier (UUID) |
| `ConvoID` | `string` | Conversation identifier grouping multiple sessions |
| `UnifiedConvoID` | `string` | Cross-conversation identifier for sub-agent chains |
| `Agent` | `*config.Agent` | Resolved agent configuration |
| `history` | `[]AgentMessage` | Conversation history (chronological message list) |
| `Type` | `string` | `SessionTypeInference` or `SessionTypeChatTurn` |
| `TTL` | `string` | Session cache TTL (default: `72h`) |
| `ToolCalls` | `[]AgentToolCall` | Pending tool calls from the model |
| `ToolResults` | `[]map[string]interface{}` | Collected tool call results |
| `ParentSessionID` | `string` | Parent session ID (set for sub-agent sessions) |

Sessions are stored in the distributed cache under the key prefix `agent_session:<ID>`.

### ConvoState

`ConvoState` (`internal/agent/convo_state.go`) is the **persisted, queryable view of a conversation**. While an `AgentSession` tracks a single inference turn, the `ConvoState` spans the entire conversation.

| Field | Type | Description |
|---|---|---|
| `ConvoID` | `string` | Canonical conversation ID |
| `UnifiedConvoID` | `string` | Shared ID across sub-agent conversations |
| `FirstSession` | `*ConvoSessionRef` | First session in this conversation |
| `LastSession` | `*ConvoSessionRef` | Most recent session (UI-visible) |
| `ActiveSession` | `*ConvoSessionRef` | Currently active session (for cancel targeting) |
| `Cancelled` | `bool` | Whether the conversation has been cancelled |
| `TotalTokens` | `int` | Cumulative input+output tokens |
| `Generation` | `int` | Number of times compaction has occurred |
| `ArchivedConvos` | `[]string` | Keys of archived pre-compaction histories |
| `Agent` | `*config.Agent` | Resolved agent configuration (shared across sessions) |
| `Skills` | `map[string]string` | Loaded skill name → path mapping |

Sessions in a conversation go through status transitions:

```
active → complete
        → failed
        → cancelled
```

ConvoState is stored under key prefix `convo_state:<ConvoID>` and is also published to a stream (`convo_stream_`) for UI consumption.

A `ConvoSessionRef` records a compact summary of each session:

```go
type ConvoSessionRef struct {
    SessionID    string
    AgentName    string
    Type         string       // "inference" or "chat_turn"
    Status       string       // "active", "complete", "failed", "cancelled"
    ErrorReason  string
    InputTokens  int
    OutputTokens int
    TotalTokens  int
    CreatedAt    time.Time
    UpdatedAt    time.Time
}
```

### PersistentAgentStore


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [honeydipper/honeydipper](https://github.com/honeydipper/honeydipper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
