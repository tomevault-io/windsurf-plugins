---
trigger: always_on
description: This document describes the planned multi-agent system for 20x. It is not yet implemented — this serves as the architectural specification for Phase 3 development.
---

# AGENTS.md — Multi-Agent Architecture (Phase 3)

This document describes the planned multi-agent system for 20x. It is not yet implemented — this serves as the architectural specification for Phase 3 development.

## Overview

20x will support running multiple AI coding agents in parallel, each working on assigned tasks within specific codebases. Agents are managed through the OpenCode SDK and interact with users via streaming terminal transcripts with human-in-the-loop (HITL) approval flows.

## Agent Model

Each agent is a persistent configuration stored in SQLite:

```typescript
interface Agent {
  id: string                    // cuid2
  name: string                  // e.g. "Backend Agent", "Frontend Agent"
  server_url: string            // OpenCode server URL
  config: AgentConfig           // stored as JSON
  is_default: boolean           // one agent is pre-seeded on first launch
  created_at: string
  updated_at: string
}

interface AgentConfig {
  model: string                 // e.g. "claude-sonnet-4-5-20250929"
  mcp_servers: McpServerConfig[]
  system_prompt?: string
  max_tokens?: number
  temperature?: number
}

interface McpServerConfig {
  name: string
  command: string
  args: string[]
  env?: Record<string, string>
}
```

## Database Schema

```sql
CREATE TABLE agents (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  server_url TEXT NOT NULL DEFAULT 'http://localhost:3000',
  config TEXT NOT NULL DEFAULT '{}',
  is_default INTEGER NOT NULL DEFAULT 0,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

-- Tasks table gets a new nullable column:
ALTER TABLE tasks ADD COLUMN agent_id TEXT REFERENCES agents(id) ON DELETE SET NULL;
```

A default agent is seeded on first launch with sensible defaults.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Renderer Process                       │
│                                                           │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────┐  │
│  │ Agent Panel  │  │ Task Detail  │  │ Agent Settings │  │
│  │ (streaming   │  │ (assign      │  │ (CRUD agents,  │  │
│  │  transcript) │  │  agent)      │  │  MCP config)   │  │
│  └──────┬───────┘  └──────┬───────┘  └───────┬────────┘  │
│         │                 │                   │           │
│         └────────┬────────┴───────────────────┘           │
│                  │ IPC                                     │
├──────────────────┼────────────────────────────────────────┤
│                  │       Main Process                      │
│                  ▼                                         │
│  ┌───────────────────────┐  ┌──────────────────────────┐  │
│  │   Agent Manager       │  │   Database Manager       │  │
│  │                       │  │                           │  │
│  │ - OpenCode SDK client │  │ - agents table CRUD      │  │
│  │ - Session lifecycle   │  │ - task-agent assignment   │  │
│  │ - Event streaming     │  │                           │  │
│  │ - HITL flow           │  │                           │  │
│  └───────────┬───────────┘  └──────────────────────────┘  │
│              │                                             │
└──────────────┼─────────────────────────────────────────────┘
               │
               ▼
  ┌────────────────────────┐
  │   OpenCode Server(s)   │
  │                        │
  │  - LLM inference       │
  │  - MCP tool execution  │
  │  - File system access  │
  └────────────────────────┘
```

## IPC Channels

### Agent CRUD

| Channel | Direction | Payload | Response |
|---------|-----------|---------|----------|
| `agent:getAll` | renderer -> main | — | `Agent[]` |
| `agent:get` | renderer -> main | `id` | `Agent` |
| `agent:create` | renderer -> main | `CreateAgentData` | `Agent` |
| `agent:update` | renderer -> main | `id, UpdateAgentData` | `Agent` |
| `agent:delete` | renderer -> main | `id` | `boolean` |

### Agent Sessions

| Channel | Direction | Payload | Response |
|---------|-----------|---------|----------|
| `agent:start` | renderer -> main | `agentId, taskId` | `{ sessionId }` |
| `agent:stop` | renderer -> main | `sessionId` | `{ success }` |
| `agent:send` | renderer -> main | `sessionId, message` | — |
| `agent:output` | main -> renderer | `{ sessionId, data }` | — |
| `agent:status` | main -> renderer | `{ agentId, status }` | — |
| `agent:approval` | main -> renderer | `{ sessionId, action, description }` | — |
| `agent:approve` | renderer -> main | `{ sessionId, approved }` | — |

## Agent Manager

`src/main/agent-manager.ts` — the core orchestration layer.

```typescript
class AgentManager extends EventEmitter {
  // Active sessions indexed by session ID
  private sessions: Map<string, AgentSession>

  // Start an agent working on a task
  async startSession(agentId: string, taskId: string): Promise<string>

  // Stop a running session
  async stopSession(sessionId: string): Promise<void>

  // Send user input to a session (for HITL approval or follow-up prompts)
  async sendMessage(sessionId: string, message: string): Promise<void>

  // Clean up all sessions on app exit
  stopAllSessions(): void
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peakflo/20x](https://github.com/peakflo/20x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
