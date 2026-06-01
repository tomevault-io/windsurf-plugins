---
trigger: always_on
description: Keep every implementation as small and obvious as possible.
---

# AGENTS.md - Giselle Development Guide

## Development Philosophy

### Core Principle: **Less is more**
Keep every implementation as small and obvious as possible.

### Guidelines
- **Simplicity first** – Prefer the simplest data structures and APIs that work
- **Avoid needless abstractions** – Refactor only when duplication hurts
- **Remove dead code early** – `pnpm tidy` scans for unused files/deps and lets you delete them in one command
- **Minimize dependencies** – Before adding a dependency, ask "Can we do this with what we already have?"
- **Consistency wins** – Follow existing naming and file-layout patterns; if you must diverge, document why
- **Explicit over implicit** – Favor clear, descriptive names and type annotations over clever tricks
- **Fail fast** – Validate inputs, throw early, and surface actionable errors
- **Let the code speak** – If you need a multi-paragraph comment, refactor until intent is obvious

## Project Overview

Giselle is built to design and run AI workflows beyond prompt chains. Not a chat. Not a chain. A system you can run.

### Key Features:

- Visual editor
- Instant execution
- No infra headaches
- Open source — self-host or use our cloud

## Architecture

### Monorepo Structure

Giselle uses a **Turborepo monorepo** with pnpm workspaces, organized into four main directories:

```
/workspace
├── apps/                    # Deployable applications
│   ├── studio.giselles.ai/  # Giselle Cloud (production)
│   └── ui.giselles.ai/      # UI component showcase
├── packages/                # Published SDK packages (@giselles-ai/*)
├── internal-packages/       # Internal shared packages (@giselle-internal/*)
└── tools/                   # Development utilities
```

### Package Layers

**SDK Packages (`packages/@giselles-ai/*`):**
- `protocol` — Core domain types and schemas (Workspace, Node, Task, Generation)
- `giselle` — Engine implementation (tasks, generations, triggers, integrations)
- `react` — React hooks and components for client integration
- `nextjs` — Next.js integration with route handlers
- `language-model` — Language model abstractions and cost calculations
- `language-model-registry` — Provider-specific model implementations
- `rag` — RAG pipeline (chunking, embedding, querying)
- `github-tool` — GitHub integration utilities

**Internal Packages (`internal-packages/@giselle-internal/*`):**
- `workflow-designer-ui` — Visual workflow editor (React Flow-based)
- `ui` — Shared UI components (Radix-based)

### Technology Stack

| Layer | Technology |
|-------|------------|
| Runtime | Node.js 22+ |
| Package Manager | pnpm 10+ |
| Build System | Turborepo |
| Web Framework | Next.js 16 (App Router) |
| UI Library | React 19 |
| Styling | Tailwind CSS 4 |
| State Management | Zustand (editor), SWR (data fetching) |
| Validation | Zod v4 |
| Database | PostgreSQL with Drizzle ORM |
| Vector Store | pgvector |
| Formatting/Linting | Biome |
| Testing | Vitest |
| AI SDK | Vercel AI SDK |

### Data Flow

```
Workspace (JSON) → Protocol Types → Giselle Engine → Task Execution → Generation Output
                                          ↓
                          Language Model Registry → AI Provider APIs
```

### Key Domain Concepts

- **Workspace** — A visual workflow containing nodes and connections
- **Node** — Either an OperationNode (actions, text generation, triggers) or VariableNode (text, files, vector stores)
- **Task** — An executable instance of a workflow with sequences of generations
- **Generation** — A single step execution (created → queued → running → completed/failed)
- **App** — A published workflow entry point with parameters

## Development Workflow

TBD
### Initial Setup

```sh
pnpm install        # Install all dependencies
pnpm build-sdk      # Build SDK packages (required before running apps)
```

### Development Commands

```sh
# Development
pnpm dev:studio.giselles.ai  # Start Giselle Cloud

# Build
pnpm build-sdk               # Build SDK packages
pnpm -F studio.giselles.ai build  # Build Giselle Cloud

# Quality Checks
pnpm format                  # Format code with Biome
pnpm check-types             # Type-check all packages
pnpm test                    # Run all tests
pnpm tidy                    # Find unused files/dependencies
pnpm tidy --fix              # Remove unused files/dependencies
```

### After Every Code Change

Run these commands in order:
1. `pnpm format` — Format code
2. `pnpm build-sdk` — Rebuild SDK packages
3. `pnpm check-types` — Verify types
4. `pnpm tidy` — Check for unused code
5. `pnpm test` — Run tests
6. Update `.continuity/` per-branch ledger — Reflect the change immediately

### API addition rule (Giselle ↔ HTTP)

When adding a new **public API** to `packages/giselle/src/giselle.ts`, also add the corresponding routing entry to `packages/http/src/router.ts` (typically `jsonRoutes.<name>` using `giselle.<name>.inputSchema`) so the API is reachable through the HTTP layer (e.g., via `NextGiselle`).

### Testing

```sh
pnpm test                           # Run all tests
pnpm -F @giselles-ai/giselle test   # Run tests for a specific package
cd packages/giselle && vitest       # Run tests in watch mode
vitest run src/tasks/run-task.test.ts  # Run a specific test file
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giselles-ai/giselle](https://github.com/giselles-ai/giselle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
