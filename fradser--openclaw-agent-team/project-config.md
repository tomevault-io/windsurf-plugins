---
trigger: always_on
description: Manages team directories under `~/.openclaw/teams/` (or custom `teamsDir`).
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a pnpm monorepo for `@fradser/openclaw-agent-team`, an OpenClaw plugin enabling multi-agent team coordination with inter-agent messaging.

**Package**: `@fradser/openclaw-agent-team` v1.0.0
**Author**: Frad LEE <fradser@gmail.com>
**License**: MIT
**Peer dependency**: `openclaw >=2026.3.2`

## Commands

Run all commands from the repository root unless noted otherwise.

```bash
# Install dependencies
pnpm install

# Build (compiles TypeScript and copies plugin manifest to dist/)
pnpm build

# Run all tests (once)
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run a single test file (from packages/openclaw-agent-team/)
pnpm vitest run tests/ledger.test.ts

# Lint
pnpm lint
```

## Repository Structure

```
openclaw-agent-team/
├── package.json                  # Monorepo root (pnpm workspace)
├── pnpm-workspace.yaml           # Workspace config: packages/*
├── CLAUDE.md                     # This file
├── docs/
│   └── plans/                    # Design and planning documents
└── packages/
    └── openclaw-agent-team/      # Main plugin package
        ├── package.json
        ├── tsconfig.json
        ├── vitest.config.ts
        ├── openclaw.plugin.json  # Plugin manifest (copied to dist/ on build)
        ├── index.ts              # Re-exports from src/index.ts
        ├── src/                  # Source files
        └── tests/                # Test files
```

## Architecture

The plugin follows a 4-layer architecture with dependencies pointing strictly inward:

```mermaid
graph TB
    subgraph Tools["Tools Layer"]
        TeamCreate[team-create.ts]
        TeamShutdown[team-shutdown.ts]
        TeammateSpawn[teammate-spawn.ts]
    end

    subgraph Core["Core Layer"]
        Index[index.ts]
        Ledger[ledger.ts]
        Channel[channel.ts]
        Runtime[runtime.ts]
        ContextInjection[context-injection.ts]
        DynamicTeammate[dynamic-teammate.ts]
    end

    subgraph Storage["Storage Layer"]
        Storage[storage.ts]
    end

    subgraph Foundation["Foundation Layer"]
        Types[types.ts]
    end

    Tools --> Core
    Core --> Storage
    Storage --> Foundation
```

### Source Modules

#### `src/index.ts` — Plugin Entry Point
Registers the plugin with OpenClaw. Responsibilities:
- Exports `PLUGIN_ID`, `PLUGIN_NAME`, `PLUGIN_DESCRIPTION` constants
- Registers 3 agent tools: `team_create`, `team_shutdown`, `teammate_spawn`
- Registers the `agent-team` channel plugin for inter-agent messaging
- Registers the `before_prompt_build` hook for teammate context injection
- Calls `syncTeammatesToConfig()` on startup to repair missing agent bindings

#### `src/types.ts` — TypeBox Schema Definitions
All types are defined using TypeBox for runtime validation. Key types:
- **`TeamConfig`** — Team metadata: `id`, `team_name`, `description`, `agent_type`, `lead`, `metadata` (createdAt, updatedAt, status)
- **`TeammateDefinition`** — Teammate record: `name`, `agentId`, `sessionKey`, `agentType`, optional `model`/`tools`, `status` (`idle`/`working`/`error`/`shutdown`), `joinedAt`
- **`AgentTeamConfig`** — Plugin configuration: `maxTeammatesPerTeam`, `defaultAgentType`, optional `teamsDir`, `pathTemplates`

Helper functions: `buildTeammateAgentId()`, `parseTeammateAgentId()`
Constants: `TEAMMATE_AGENT_ID_PREFIX`, `AGENT_TEAM_CHANNEL`, `DEFAULT_WORKSPACE_TEMPLATE`, `DEFAULT_AGENT_DIR_TEMPLATE`
Validation: `validateTeamConfig()`, `validateTeammateDefinition()`, `validateAgentTeamConfig()`

#### `src/ledger.ts` — JSONL Member Persistence
`TeamLedger` class using JSONL files with in-memory caches for teammate member tracking. Lazy-loads on first access.

JSONL files (in team directory):
- `members.jsonl` — Teammate member records (sessionKey, name, agentId, agentType, status, joinedAt)

Member methods: `addMember()`, `listMembers()`, `updateMemberStatus()`, `removeMember()`

**Note**: Task tracking functionality is not implemented. The ledger only handles teammate member persistence.

#### `src/storage.ts` — File System Operations
Manages team directories under `~/.openclaw/teams/` (or custom `teamsDir`).

Exports:
- `TEAM_NAME_PATTERN`, `TEAMMATE_NAME_PATTERN` — validation regexes
- `validateTeamName()`, `sanitizeTeammateName()` — name sanitization
- `createTeamDirectory()`, `teamDirectoryExists()`, `deleteTeamDirectory()` — directory lifecycle
- `writeTeamConfig()`, `readTeamConfig()` — JSON config I/O
- `resolveTeammatePaths()`, `resolveTeamPath()`, `getTeamsBaseDir()` — path helpers
- `resolveTemplatePath()` — template-based path resolution

#### `src/channel.ts` — Inter-Agent Messaging Channel
Implements the OpenClaw `ChannelPlugin` interface as the `agent-team` channel.

- Target format: `"teamName:teammateName"` (resolved via `normalizeTarget`)
- Messages written as JSONL to: `{teamsDir}/{teamName}/inbox/{teammateName}/messages.jsonl`
- Capabilities: direct messaging, reply; no polls/threads/media/reactions/edit
- Single hardcoded `"default"` account, always enabled

**Outbound messaging:**
- `resolveTarget()` — validates target format and returns normalized target
- `sendText()` — sends text messages to teammate inbox

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FradSer/openclaw-agent-team](https://github.com/FradSer/openclaw-agent-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
