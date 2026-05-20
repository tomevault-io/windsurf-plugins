---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ChainGraph is a flow-based programming framework for building visual computational graphs. It's a **pnpm monorepo** using **Turbo** for task orchestration, **TypeScript** throughout, and **Bun** as the development runtime.

## Project Structure

### Applications (apps/)
- **chaingraph-backend**: Legacy tRPC server (in-memory/simple storage)
- **chaingraph-frontend**: React+Vite visual flow editor using XYFlow
- **chaingraph-execution-api**: Scalable tRPC API server for execution management
- **chaingraph-execution-worker**: Worker service for processing flow executions

### Core Packages (packages/)
- **chaingraph-types**: Foundation package with port system, decorators, base node classes, flow definitions
- **chaingraph-nodes**: Pre-built node implementations (AI, flow control, data manipulation, etc.)
- **chaingraph-trpc**: tRPC layer with real-time subscriptions (WebSocket), database schemas (Drizzle ORM)
- **chaingraph-executor**: **THE EXECUTION ENGINE** - Contains DBOS workflows, execution services, event streaming
- **badai-api**: BadAI platform integration (GraphQL client)

### Supporting Packages
- **badai-api-example**: Example usage of BadAI API client
- **typescript-config**: Shared TypeScript configurations

### Development Tools
- **.claude/skills/**: 15 Claude Code skill modules for AI-assisted development
- **.conductor/**: DBOS Conductor runtime directory

## Claude Code Skills

The `.claude/skills/` directory contains 15 skill modules that provide context-aware documentation for AI-assisted development. Skills are automatically loaded based on trigger keywords.

### Meta Skills
| Skill | Purpose |
|-------|---------|
| `skill-authoring` | Guidelines for creating new Claude Code skills |
| `skill-maintenance` | When and how to update skills after codebase changes |

### Foundation
| Skill | Purpose |
|-------|---------|
| `chaingraph-concepts` | Core domain concepts - flows, nodes, ports, edges, execution lifecycle, event types |

### Package Architecture
| Skill | Purpose |
|-------|---------|
| `frontend-architecture` | React/Vite frontend structure, Effector stores, providers, components |
| `executor-architecture` | Execution engine, DBOS workflows, services, event bus, task queues |
| `types-architecture` | Foundation types, decorators, port system, flow definitions |

### Technology Patterns
| Skill | Purpose |
|-------|---------|
| `effector-patterns` | Effector state management patterns and **anti-patterns to avoid** |
| `dbos-patterns` | DBOS durable execution **constraints** - what's allowed in workflows vs steps |
| `xyflow-patterns` | XYFlow integration - custom nodes/edges, handles, drag-drop, performance |
| `trpc-patterns` | tRPC framework - routers, procedures, subscriptions, middleware, WebSocket |

### Feature Skills
| Skill | Purpose |
|-------|---------|
| `port-system` | 9 port types, plugins, factory, transfer rules, validation |
| `subscription-sync` | Real-time WebSocket subscriptions, event buffering, race condition solutions |
| `optimistic-updates` | Optimistic UI, echo detection, pending mutations, position interpolation |
| `trpc-flow-editing` | Flow editing procedures - CRUD, nodes, edges, ports, locking |
| `trpc-execution` | Execution procedures - create/start/pause/resume/stop, signal pattern |

### Using Skills

Skills are triggered automatically by keywords in your prompts. Examples:
- "How do I create a new node?" → triggers `chaingraph-concepts`, `types-architecture`
- "Fix the DBOS workflow" → triggers `dbos-patterns`, `executor-architecture`
- "Add optimistic update for port value" → triggers `optimistic-updates`, `effector-patterns`

## Common Commands

### Development
```bash
# Install dependencies
pnpm install

# Run all services (frontend + backend/execution-api)
pnpm run dev

# Run individual services
pnpm run dev:front              # Frontend only
pnpm run dev:back               # Backend only
pnpm run dev:execution-worker   # Execution worker only

# Build everything
pnpm run build

# Build specific apps
pnpm run build:front
pnpm run build:back
```

### Testing
```bash
# Run all tests
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run tests with coverage
pnpm test:coverage

# Run tests with UI
pnpm test:ui

# Run specific package tests
pnpm --filter @badaitech/chaingraph-types test
pnpm --filter @badaitech/chaingraph-executor test
```

### Code Quality
```bash
# Type check all packages
pnpm run typecheck

# Lint
pnpm run lint

# Lint and auto-fix
pnpm run lint:fix
```

### Database Management
```bash
# Start PostgreSQL (Docker)
docker compose up -d postgres

# Run migrations (creates schema in PostgreSQL)
pnpm run migrate

# Generate new migration (after schema changes)
DATABASE_URL="postgres://postgres@0.0.0.0:5432/postgres?sslmode=disable" npm run migrate:generate
```

### Docker
```bash
# Build Docker images
pnpm run docker:backend
pnpm run docker:frontend
pnpm run docker:execution-api
pnpm run docker:execution-worker
pnpm run docker:build-all

# Or use Makefile
make docker-build-backend
make docker-build-frontend
make docker-build-all


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaingraphlabs/chaingraph](https://github.com/chaingraphlabs/chaingraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
