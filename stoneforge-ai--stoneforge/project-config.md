---
trigger: always_on
description: Context and instructions for AI coding agents working on the Stoneforge repository.
---

# AGENTS.md

Context and instructions for AI coding agents working on the Stoneforge repository.

## How to Find Documentation

1. **This file** — Quick task-to-file mappings below
2. **Documentation Directory** — Full index of all workspace docs: `sf show el-30yb`
3. **Category Index** — Browse by category: `sf show el-og6v`
4. **Full-text search** — Find any doc by topic: `sf document search "your topic"`

---

## Quick Start

### Backend Tasks

| I need... | Key Files | Reference Doc |
|-----------|-----------|---------------|
| Add an API endpoint | `apps/quarry-server/src/index.ts`, `apps/smithy-server/src/routes/` | `sf show el-5z1q` |
| Add a core type | `packages/core/src/types/` | `sf show el-6c3s` |
| Work with dependencies | `packages/quarry/src/services/dependency.ts` | `sf show el-200z` |
| Add an orchestrator service | `packages/smithy/src/services/` | `sf show el-50ia` |
| Use the CLI | `packages/quarry/src/cli/commands/` | `sf show el-59tr` |
| Use the QuarryAPI | `packages/quarry/src/api/quarry-api.ts` | `sf show el-kflh` |
| Use the OrchestratorAPI | `packages/smithy/src/api/orchestrator-api.ts` | `sf show el-3qg2` |
| Customize agent prompts | `.stoneforge/prompts/` | `sf show el-32rb` |
| Configure identity | `packages/quarry/src/systems/identity.ts` | `sf show el-2jw5` |
| Understand event sourcing | `packages/core/src/types/event.ts` | `sf show el-58k3` |
| Configure the system | `packages/quarry/src/config/` | `sf show el-z1sj` |

### Frontend Tasks

| I need... | Key Files | Reference Doc |
|-----------|-----------|---------------|
| Modify a smithy-web page | `apps/smithy-web/src/routes/{page}/` | `sf show el-4b3q` |
| Modify a quarry-web page | `apps/quarry-web/src/routes/{page}/` | `sf show el-4iiz` |
| Add a new frontend route | `apps/{app}/src/router.tsx` | `sf show el-232d` |
| Modify shared UI components | `packages/ui/src/` | `sf show el-2hk5` |
| Add a React component | `apps/{app}/src/components/` | `sf show el-3gg7` |
| Work with API hooks (smithy) | `apps/smithy-web/src/api/hooks/` | `sf show el-4b3q` |
| Work with API hooks (quarry) | `apps/quarry-web/src/api/hooks/` | `sf show el-4iiz` |
| Understand frontend architecture | — | `sf show el-935d` |

---

## Repository Structure

```
packages/
├── core/              # @stoneforge/core - types, errors, ID generation
├── storage/           # @stoneforge/storage - SQLite backends (Bun, Node, Browser)
├── quarry/            # @stoneforge/quarry - QuarryAPI, services, sync, CLI
├── ui/                # @stoneforge/ui - React components, hooks, design tokens
├── shared-routes/     # @stoneforge/shared-routes - HTTP route factories
└── smithy/            # @stoneforge/smithy - agent orchestration

apps/
├── quarry-server/     # Platform HTTP + WebSocket (port 3456)
├── quarry-web/        # Platform React SPA (port 5173)
├── smithy-server/     # Orchestrator API (port 3457)
├── smithy-web/        # Orchestrator dashboard (port 5174)
├── docs/              # @stoneforge/docs - Astro documentation site
└── website/           # @stoneforge/website - Public website

.stoneforge/            # Project data (config.yaml, sync/, prompts/, uploads/)
```

### Package Dependency Graph

```
@stoneforge/core           (shared types, no dependencies)
       ↓
@stoneforge/storage        (SQLite backends)
       ↓
@stoneforge/shared-routes  (HTTP route factories)
       ↓
@stoneforge/quarry         (API, services, sync, CLI)
       ↓
@stoneforge/smithy         (agent orchestration)

@stoneforge/ui             (React components, hooks — depends on core, shared-routes)
```

---

## Core Concepts

### Element Types

- **Core Types**: Task, Message, Document, Entity
- **Collection Types**: Plan, Workflow, Playbook, Channel, Library, Team
- **All inherit from Element** (id, type, timestamps, tags, metadata, createdBy)

### Dual Storage Model

- **SQLite**: Fast queries, indexes, FTS - the **cache**
- **JSONL**: Git-tracked, append-only - the **source of truth**

### Dependencies

- **Blocking types**: `blocks`, `awaits`, `parent-child` - affect task status
- **Non-blocking**: `relates-to`, `mentions`, `references` - informational only
- `blocked` status is **computed** from dependencies, never set directly

### Agent Roles (Orchestrator)

- **Director**: Owns task backlog, spawns workers, makes strategic decisions
- **Worker**: Executes assigned tasks (ephemeral or persistent)
- **Steward**: Handles code merges, documentation scanning and fixes

---

## Development Workflow

### Build & Test

```bash
bun install           # Install dependencies
bun run build         # Build all packages
bun test              # Run test suite
bun test --watch      # Watch mode
```

### CLI Usage

```bash
sf task ready         # List ready tasks
sf task blocked       # List blocked tasks
sf show <id>          # Show element details
sf task create --title "..." --priority 3 --type feature
sf dependency add --type=blocks <blocked-id> <blocker-id>
sf task close <id> --reason "..."
sf stats              # View progress stats
```

### Running Apps

```bash
bun run --filter @stoneforge/quarry-server dev       # Platform server (port 3456)
bun run --filter @stoneforge/quarry-web dev          # Platform web (port 5173)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stoneforge-ai/stoneforge](https://github.com/stoneforge-ai/stoneforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
