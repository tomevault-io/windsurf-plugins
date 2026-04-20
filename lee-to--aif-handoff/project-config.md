---
trigger: always_on
description: > Project map for AI agents. Keep this file up-to-date as the project evolves.
---

# AGENTS.md

> Project map for AI agents. Keep this file up-to-date as the project evolves.

## Project Overview

Autonomous task management system with Kanban board and AI subagents. Tasks flow through stages automatically (Backlog → Planning → Plan Ready → Implementing → Review → Done), each handled by runtime-resolved subagent workflows (Claude adapter first).

## Tech Stack

- **Language:** TypeScript (ES2022, ESNext modules)
- **Monorepo:** Turborepo (npm workspaces)
- **API:** Hono + WebSocket
- **Runtime Abstraction:** `@aif/runtime` workspace (runtime/provider contracts + registry)
- **Database:** SQLite (better-sqlite3 + drizzle-orm)
- **Frontend:** React 19 + Vite + TailwindCSS 4
- **Runtime:** Pluggable adapter system (`@aif/runtime`) — built-in Claude (Agent SDK) + Codex (CLI/API) + OpenRouter (API) adapters
- **Agent:** Runtime-neutral coordinator + node-cron
- **Testing:** Vitest

## Project Structure

```
packages/
├── shared/              # @aif/shared — contracts, schema, state machine, env, constants, logger
│   └── src/
│       ├── schema.ts        # Drizzle ORM schema (SQLite)
│       ├── types.ts         # Shared TypeScript types + RuntimeTransport enum
│       ├── stateMachine.ts  # Task stage transitions
│       ├── constants.ts     # App constants
│       ├── env.ts           # Environment validation
│       ├── logger.ts        # Pino logger setup
│       ├── index.ts         # Node exports
│       └── browser.ts       # Browser-safe exports
├── runtime/             # @aif/runtime — runtime/provider contracts, registry, validation/discovery services, adapters
│   └── src/
│       ├── index.ts         # Public API exports
│       ├── types.ts         # RuntimeAdapter interface, capabilities, execution intent
│       ├── registry.ts      # RuntimeRegistry — adapter registration and lookup
│       ├── bootstrap.ts     # Factory: create registry with built-in adapters
│       ├── resolution.ts    # Profile resolution (task → project → system → env fallback)
│       ├── readiness.ts     # Health check across all registered runtimes
│       ├── capabilities.ts  # Capability assertion before workflow execution
│       ├── promptPolicy.ts  # Agent definition vs slash-command fallback
│       ├── workflowSpec.ts  # Workflow kind, session reuse, required capabilities
│       ├── modelDiscovery.ts # Model listing + connection validation with cache
│       ├── cache.ts         # Generic in-memory TTL cache
│       ├── trust.ts         # Opaque Symbol-based trust token for permission bypass
│       ├── errors.ts        # Runtime error hierarchy
│       ├── module.ts        # Dynamic module loader for external adapters
│       └── adapters/
│           ├── TEMPLATE.ts      # Adapter development guide + skeleton
│           ├── claude/          # Claude adapter (Agent SDK transport)
│           ├── codex/           # Codex adapter (CLI + API transports)
│           └── openrouter/      # OpenRouter adapter (API transport)
├── data/                # @aif/data — centralized data-access layer
│   └── src/
│       └── index.ts         # Repository-style DB operations for API/Agent
├── api/                 # @aif/api — Hono REST + WebSocket server (port 3009)
│   └── src/
│       ├── index.ts         # Server entry point
│       ├── routes/          # tasks.ts, projects.ts, chat.ts, runtimeProfiles.ts
│       ├── services/        # runtime.ts, fastFix.ts, roadmapGeneration.ts
│       ├── middleware/      # logger.ts, rateLimit.ts, zodValidator.ts
│       ├── schemas.ts       # Zod request validation
│       └── ws.ts            # WebSocket handler
├── web/                 # @aif/web — React Kanban UI (port 5180)
│   └── src/
│       ├── App.tsx          # Root component
│       ├── components/
│       │   ├── kanban/      # Board, Column, TaskCard, AddTaskForm
│       │   ├── task/        # TaskDetail, TaskPlan, TaskLog, AgentTimeline
│       │   ├── layout/      # Header, CommandPalette
│       │   ├── project/     # ProjectSelector, ProjectRuntimeSettings
│       │   ├── settings/    # RuntimeProfileForm
│       │   └── ui/          # Reusable UI primitives (badge, button, dialog, etc.)
│       ├── hooks/           # useTasks, useProjects, useWebSocket, useTheme, useRuntimeProfiles
│       └── lib/             # api.ts, notifications.ts, utils.ts
└── agent/               # @aif/agent — Coordinator + runtime-driven subagent orchestration
    └── src/
        ├── index.ts         # Agent entry point
        ├── coordinator.ts   # Polling coordinator (node-cron)
        ├── subagentQuery.ts # Universal runtime-backed query execution
        ├── reviewGate.ts    # Auto-review gate using adapter lightModel
        ├── hooks.ts         # Activity logging, project root
        ├── stderrCollector.ts # Generic stderr ring-buffer
        ├── notifier.ts      # Notification system
        ├── codex/           # Codex login broker (OAuth-in-Docker bridge)
        └── subagents/       # planner.ts, implementer.ts, reviewer.ts

.claude/agents/          # Agent definitions (loaded by runtimes that support them)
.docker/                 # Dockerfile, entrypoint, Angie configs
data/                    # SQLite database files (gitignored)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lee-to/aif-handoff](https://github.com/lee-to/aif-handoff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
