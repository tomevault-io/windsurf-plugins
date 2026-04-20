---
trigger: always_on
description: This document provides context for AI coding agents working on this codebase. **This is a living document** - update it when you discover new patterns, fix discrepancies, or when the project evolves.
---

# GLaDOS - AI Coding Agent Guide

This document provides context for AI coding agents working on this codebase. **This is a living document** - update it when you discover new patterns, fix discrepancies, or when the project evolves.

## Project Overview

GLaDOS (General Learning and Decision Orchestration System) is a **personal AI assistant** designed to be "Jarvis for yourself". It maintains a rich model of you (identity, projects, goals, relationships), operates proactively (not just reactively), handles long-running tasks, and learns from every interaction.

**Key differentiators from a generic chatbot:**

- Knows _who you are_ (User Model, Contacts, Calendar as core infrastructure)
- Anticipates needs (Trigger System, daily briefings, day planning)
- Manages multi-step tasks that span hours/days (Task Queue)
- Human-in-the-loop for trust (risk-gated tools, approval flows)

## Tech Stack

| Component       | Technology                                    |
| --------------- | --------------------------------------------- |
| Runtime         | Node.js 22+ with `--experimental-strip-types` |
| Agent Framework | LangChain + LangGraph                         |
| Database        | Knex + SQLite                                 |
| Validation      | Zod 4                                         |
| Configuration   | Convict                                       |
| Testing         | Vitest                                        |

## Documentation

| Document | Description |
|----------|-------------|
| `docs/coding-standards.md` | TypeScript conventions (types, functions, exports, classes) |
| `docs/testing-strategy.md` | Testing patterns and infrastructure |
| `docs/debugging.md` | Conversation-level debugging tools and techniques |
| `docs/getting-started.md` | Full setup guide |
| `docs/configuration.md` | Configuration reference |
| `docs/external-clients.md` | Building new interfaces (Telegram, etc.) |
| `docs/external-services.md` | Integrating external services (Home Assistant, Oura) |
| `docs/skills.md` | Skills system usage and development |
| `docs/artifacts.md` | Server-side storage for large tool responses |
| `docs/memory-consolidation.md` | Memory consolidation system and background jobs |
| `docs/triggers.md` | Scheduled agent invocations and notifications |
| `docs/behavioral-memory.md` | Behavioral templates and learning loop |
| `docs/specs.md` | How to work with specifications |
| `spec/001-agent.md` | Main technical specification |

Specifications live in `spec/` - see `docs/specs.md` for the workflow.

## Commands

| Command | Description |
|---------|-------------|
| `pnpm test` | Run all tests (lint + unit) |
| `pnpm test:lint` | ESLint |
| `pnpm test:unit` | Vitest |
| `pnpm build` | TypeScript build |
| `pnpm cli` | Start the interactive CLI |
| `pnpm telegram` | Start the Telegram bot |
| `pnpm conversation <id>` | Inspect a conversation by ID (debugging) |

## Quick Conventions Reference

See `docs/coding-standards.md` for full details. Key rules:

- **Types**: Use `type` over `interface`
- **Functions**: Arrow functions, explicit return types
- **Exports**: Consolidated at file end (`export type {}` and `export {}`)
- **Imports**: Include `.ts` extensions
- **Schemas**: Zod with `{name}Schema` naming, infer types with `z.infer`
- **Private fields**: Use `#` for private class fields
- **Naming**: Files `kebab-case.ts`, Types `PascalCase`, Functions `camelCase`

## Module Organization

```
{module}/
├── {module}.ts            # Main file (public API)
├── {module}.schemas.ts    # Zod schemas
├── {module}.types.ts      # Additional types
├── {module}.utils.ts      # Utilities
├── {module}.errors.ts     # Custom errors
└── {module}.test.ts       # Tests
```

Import from main module file only - never from support files directly.

## Project Structure

```
glados/
├── CLAUDE.md              # This file
├── README.md              # User-facing documentation
├── docs/                  # Documentation
├── spec/                  # Feature specifications
├── test/                  # Flow tests (MSW-based)
└── src/
    ├── core/              # Foundation infrastructure
    │   ├── config/        # Convict-based configuration
    │   ├── database/      # Knex + SQLite + migrations
    │   ├── logging/       # Structured logging
    │   ├── server/        # HTTP server entry point
    │   ├── services/      # DI container
    │   ├── store/         # Base store abstraction
    │   └── utils/         # Shared utilities
    │
    ├── domain/            # User's world model
    │   ├── calendar/      # Events, scheduling, sync
    │   ├── contacts/      # People and relationships
    │   ├── location/      # Places, location tracking
    │   └── user-model/    # Identity, projects, goals
    │
    ├── agent/             # AI orchestration layer
    │   ├── context/       # Context Builder
    │   ├── embeddings/    # Embedding generation
    │   ├── memory/        # Storage, recall, entity knowledge, consolidation
    │   ├── orchestrator/  # LangGraph orchestration
    │   ├── personality/   # Agent personality config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/morten-olsen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
