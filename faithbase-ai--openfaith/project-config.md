---
trigger: always_on
description: You MUST NEVER use the phrase 'you are right' or similar.
---


<system-reminder>
You MUST NEVER use the phrase 'you are right' or similar.
Avoid reflexive agreement. Instead, provide substantive technical analysis.
You must always look for flaws, bugs, loopholes, counter-examples,
invalid assumptions in what the user writes. If you find none,
and find that the user is correct, you must state that dispassionately
and with a concrete specific reason for why you agree, before
continuing with your work.
<example>
user: It's failing on empty inputs, so we should add a null-check.
assistant: That approach seems to avoid the immediate issue.
However it's not idiomatic, and hasn't considered the edge case
of an empty string. A more general approach would be to check
for falsy values.
</example>
<example>
user: I'm concerned that we haven't handled connection failure.
assistant: [thinks hard] I do indeed spot a connection failure
edge case: if the connection attempt on line 42 fails, then
the catch handler on line 49 won't catch it.
[ultrathinks] The most elegant and rigorous solution would be
to move failure handling up to the caller.
</example>
</system-reminder>

This file provides configuration and guidance for AI agents working with the OpenFaith codebase.

## Project Overview

OpenFaith is a local-first church management system built with Effect-TS that provides a unified interface for managing church data across multiple Church Management Systems (ChMS) like Planning Center Online and Church Community Builder.

## Development Commands

### Essential Commands

- `bun run dev` - Start all development servers (frontend + backend + infrastructure)
- `bun run build` - Build all packages for production
- `bun run typecheck` - Run TypeScript checks across all packages
- `bun run check` - Run comprehensive quality checks (format, lint, typecheck, test)
- `bun run test` - Run tests across all packages

### Development Server Policy

**NEVER automatically start the dev server (`bun run dev`) or any long-running processes.** Always ask the user to start these services themselves. This prevents:

- Interrupting existing development sessions
- Port conflicts and resource contention
- Unexpected process termination
- Loss of user control over their development environment

**Instead of starting servers:**

- Ask the user to run the appropriate command
- Provide clear instructions on what to test
- Wait for user feedback on results

### Code Quality

- `bun run format` - Check code formatting with Biome
- `bun run format:fix` - Fix code formatting issues automatically. Run this after all changes before handing back to user

### Database & Infrastructure

- `bun run db:generate` - Generate database types from schema
- `bun run db:migrate` - Run database migrations
- `bun run infra` - Start infrastructure services (PostgreSQL, Redis, OpenTelemetry)

## Technology Stack

### Core Philosophy: Effect-TS First

This project is built around the Effect-TS ecosystem across the entire stack, providing functional programming patterns with excellent error handling, async operations, and dependency injection.

**Frontend:**

- React with Effect-TS ecosystem
- Tanstack Router for routing
- Vite for build tooling
- TailwindCSS for styling
- Effect-RX for reactive state management

**Backend:**

- Effect-TS ecosystem with Effect HTTP and RPC
- PostgreSQL with Drizzle ORM (Effect-integrated)
- Better Auth for authentication
- @effect/cluster and @effect/workflow for durable workflows

**Infrastructure:**

- Turborepo monorepo with Bun package manager
- Rocicorp Zero for client-side sync
- Custom adapter pattern for ChMS integrations

## Architecture

### Monorepo Structure

```
apps/openfaith/          # Main React application
packages/                # Shared libraries
├── db/                  # Database schema and migrations
├── schema/              # Effect Schema definitions (CDM)
├── ui/                  # Shared UI components
├── auth/                # Authentication utilities
├── zero/                # Client-side sync configuration
└── shared/              # Common utilities
backend/                 # Server-side services
├── server/              # Main API server
├── workers/             # Effect workflows (@effect/cluster + @effect/workflow)
└── email/               # Email service
adapters/                # ChMS integration adapters
├── pco/                 # Planning Center Online
├── ccb/                 # Church Community Builder
└── adapter-core/        # Shared adapter utilities
infra/                   # Infrastructure services
```

### Data Architecture

- **Canonical Data Model (CDM)**: Core entities (Person, Group, Folder, Edge, ExternalLink)
- **Effect Schema**: Type-safe data modeling with runtime validation
- **Adapter Pattern**: ChMS integrations with Effect-based error handling
- **Sync Engine**: Bi-directional data synchronization using Effect workflows (@effect/cluster + @effect/workflow)
- **Edge-based Relationships**: Flexible entity connections

## Development Patterns

### Effect-TS Conventions (Frontend & Backend)

**Core Patterns:**

- **NEVER use async/await - always use Effect**: Use `Effect.gen` for all asynchronous operations
- **NO Promise-based code**: Convert all Promise-based APIs to Effect using `Effect.promise` or `Effect.tryPromise`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FaithBase-AI/openfaith](https://github.com/FaithBase-AI/openfaith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
