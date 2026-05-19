---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Common Development Commands

### Package Management

- `pnpm install` - Install all dependencies (uses pnpm workspaces)
- `./scripts/build.sh` - Build all packages
- `./scripts/build.sh --watch` - Build packages in watch mode
- `./scripts/clean.sh` - Clean build artifacts
- `./scripts/clean.sh --dry-run` - Preview what would be cleaned

### Testing

- `bun run packages/core` - Run core package tests (primary test command)

### Documentation

- `cd docs && bun run dev` - Run documentation development server
- `cd docs && bun run docs:build` - Build documentation

### Code Quality

- `pnpx prettier --check packages` - Check code formatting
- `pnpx prettier --write packages` - Format code
- `knip` - Find unused dependencies and exports

## Project Architecture

This is **Daydreams**, a TypeScript framework for building stateful AI agents.
The architecture is designed around:

### Core Components (`packages/core/`)

- **Agent (`dreams.ts`)**: Main orchestrator managing lifecycle, context state,
  and execution
- **Context System (`context.ts`)**: Type-safe isolated state management for
  conversations/tasks
- **Memory System (`memory/`)**: Dual-tier storage (working memory + persistent
  storage)
- **Task Runner (`task.ts`)**: Async operation management with concurrency
  control
- **Engine (`engine.ts`)**: Execution engine processing inputs and coordinating
  actions

### Package Structure

- **`packages/core/`** - Core framework with agent, context, memory, and task
  systems
- **`packages/*`** - Extensions for platforms (discord, twitter, telegram),
  storage (supabase, chroma, mongo), chains (hyperliquid, defai), utilities
  (cli, create-agent, synthetic)
- **`examples/`** - Working examples organized by use case (basic, chains,
  games, social platforms)
- **`clients/example-ui/`** - React frontend demonstrating agent capabilities
- **`docs/`** - Next.js documentation site

### Key Concepts

- **Context**: Isolated stateful environment (like a chat session) with
  type-safe args and memory
- **Working Memory**: Temporary execution state (inputs, outputs, calls,
  results, thoughts)
- **Actions**: Type-safe functions agents can execute
- **Extensions**: Plugin architecture for platforms, storage, and custom
  features

### Memory Architecture

The system uses a two-tier memory approach:

1. **Working Memory**: Temporary state during execution (logs, calls, results)
2. **Persistent Storage**: Long-term memory via pluggable stores (KV, Vector)

Context state is automatically persisted and restored between sessions.

### Context System

Each context maintains isolated state identified by `type:key`. Contexts can
have:

- Custom creation logic (`create`)
- Schema validation for arguments (`schema`)
- Setup/teardown hooks (`setup`, `onStep`, `onRun`, `onError`)
- Custom save/load logic (`save`, `load`)

## Development Notes

### Writing Documentation

Follow `.cursor/rules/write-docs.mdc` guidelines:

- Avoid marketing language ("powerful", "built-in", "complete")
- Focus on technical details over benefits
- Address engineers directly with nuts-and-bolts information
- Use frontmatter in MDX tutorials
- Extract meaningful content from examples, not just copy code

### Monorepo Structure

- Uses **pnpm workspaces** with Lerna for package management
- TypeScript configuration shared via `tsconfig.json` at root
- Build system uses `tsup` for individual packages
- Dependencies managed via catalog pattern in `package.json`

### Testing Approach

- Primary testing runs through `bun run packages/core`
- Tests are co-located with source files (`*.test.ts`)
- Uses Vitest as the test runner

### Extension Development

New extensions should follow the pattern:

- Implement extension interface with optional `inputs`, `outputs`, `actions`,
  `services`
- Provide `install` hook for setup
- Register contexts if the extension adds new context types

## Complete Data Flow Architecture

### 1. Agent Creation and Initialization

When you create an agent with `createDreams()`:

```
createDreams(config) → Agent instance
├── Container (dependency injection)
├── TaskRunner (concurrency management)
├── Memory (store + vector)
├── Registry (contexts, actions, outputs, inputs)
└── Context tracking maps (contextIds, contexts, contextsRunning)
```

The agent maintains:

- **contextIds**: Set of all known context IDs
- **contexts**: Map of context ID → ContextState
- **contextsRunning**: Map of context ID → running execution state
- **workingMemories**: Map of context ID → WorkingMemory

### 2. Message Processing Flow

When a user sends a message via `agent.send()`:

```
1. Create InputRef → wraps user input with metadata
2. Call agent.run() with InputRef in chain
3. Get/Create ContextState for the conversation
4. Get/Create WorkingMemory for the context
5. Create Engine instance for this execution
6. Process through Engine's router system
7. Generate LLM response with structured XML parsing
8. Handle outputs and persist state
```

### 3. Engine: The Execution Core

The Engine (`engine.ts`) orchestrates all execution:

```
Engine State:
├── step: Current execution step number

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daydreamsai/daydreams](https://github.com/daydreamsai/daydreams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
