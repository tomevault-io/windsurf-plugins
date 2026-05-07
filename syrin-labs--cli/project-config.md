---
trigger: always_on
description: Enables UI updates without tight coupling. Supports both sync and async subscribers.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Syrin** is a runtime intelligence system for MCP (Model Context Protocol) servers. It helps developers validate, test, and reason about MCP execution before production deployment through static analysis, contract-based testing, and interactive development environments.

## Development Commands

### Building and Running

```bash
# Build the project (TypeScript compilation + path alias resolution + template copy)
npm run build

# Run Syrin CLI locally (rebuilds and executes)
npm run syrin <command>

# Example: Run dev mode locally
npm run syrin dev --exec
```

### Testing

```bash
# Run all tests (watch mode)
npm test

# Run tests once
npm run test:run

# Run tests with coverage
npm run test:coverage

# Run tests with UI
npm run test:ui

# Run specific test file
npm test -- src/runtime/dev/session.test.ts
```

### Code Quality

```bash
# Lint the codebase
npm run lint

# Lint and fix issues
npm run lint:fix

# Format code (Prettier)
npm run format

# Check formatting
npm run format:check

# Type check without building
npm run type-check
```

### Publishing

```bash
# Prepare for publish (runs build automatically)
npm run prepublishOnly
```

## High-Level Architecture

Syrin uses a **layered, event-driven architecture** with five primary layers:

### 1. CLI Command Layer (`/cli`)

Entry point using Commander.js. Commands map to handlers that orchestrate lower layers:
- `init`: Initialize project configuration
- `doctor`: Validate configuration and environment
- `dev`: Interactive LLM-MCP development environment
- `test`: Contract-based tool testing with sandboxing
- `analyse`: Static validation of MCP tool contracts
- `list`: Inspect tools, resources, and prompts
- `config`: Manage local/global configuration
- `update`/`rollback`: Version management

All commands use unified error handling via `command-error-handler.ts`.

### 2. Configuration Management (`/config`)

**Dual-layer configuration system:**
- **Local Config**: `syrin.yaml` in project root (all settings)
- **Global Config**: `~/.syrin/syrin.yaml` (LLM providers only)

**Key files:**
- `loader.ts`: Loads local config (project-specific)
- `global-loader.ts`: Loads global config (user-wide)
- `merger.ts`: Implements precedence (local > global > CLI flags)
- `schema.ts`: Zod validation for both config types
- `env-checker.ts`: Environment variable validation

**Precedence order:** CLI Flags > Local Config > Global Config > Defaults

### 3. Runtime Engines (`/runtime`)

Core processing subsystems:

#### MCP Client (`/runtime/mcp`)
- Manages connections to MCP servers
- `manager.ts`: HTTPMCPClientManager and StdioMCPClientManager
- `client/base.ts`: Base functionality for tool discovery/execution
- `connection.ts`: Creates MCP client instances with transport setup

#### LLM Runtime (`/runtime/llm`)
- Abstracts multiple LLM providers (OpenAI, Claude, Ollama)
- `factory.ts`: Factory pattern for provider instantiation
- `LLMProvider` interface with unified `chat()` method
- Each provider normalizes tool calls to common format

#### Dev Mode (`/runtime/dev`)
- Interactive LLM-MCP development environment
- `session.ts`: Orchestrates LLM-MCP interactions with state management
- `repl.ts`: Interactive command loop with history
- `event-mapper.ts`: Maps MCP/LLM events to runtime event types
- `data-manager.ts`: Manages session state and conversation history

#### Analysis Engine (`/runtime/analysis`)
- Static validation of MCP tools
- `analyser.ts`: Main orchestrator running all rules
- **Pipeline**: load → normalize → index → infer dependencies → run rules
- **36 validation rules** (E000-E600 errors, W100-W301 warnings)
- Each rule extends `BaseRule` and implements semantic checks

#### Test Runtime (`/runtime/test`)
- Contract-based testing with sandboxing
- `orchestrator.ts`: Coordinates test execution
- `contract-loader.ts`: Loads tool test specifications from `tools/*.yaml`
- `behavior-observer.ts`: Detects side effects, non-determinism, output explosion
- `runner.ts`: Executes tools in isolated sandbox

#### Sandbox (`/runtime/sandbox`)
- Process isolation for tool execution
- `executor.ts`: Spawns and manages MCP server processes
- `io-monitor.ts`: Captures stdout/stderr with size/time limits
- Memory and timeout enforcement for safety

### 4. Event System (`/events`)

**Persistent, typed event streaming** throughout the application:

**Event Categories (36 types across 10 categories):**
- **A**: Session lifecycle (5 events)
- **B**: Workflow/steps (5 events)
- **C**: LLM context (2 events)
- **D**: LLM proposals (3 events) - non-authoritative
- **E**: Validation (5 events) - runtime authority
- **F**: Tool execution (3 events) - ground truth
- **G**: Transport (4 events)
- **H**: Registry (4 events)
- **I**: Testing (2 events)
- **J**: Diagnostics (3 events)

**Key components:**
- `emitter.ts`: `RuntimeEventEmitter` with auto-incrementing sequence numbers
- `store.ts`: Event persistence interface
  - `MemoryEventStore`: In-process storage
  - `FileEventStore`: JSONL files in `.syrin/events/<sessionId>.jsonl`
- `payloads/`: Typed payload definitions for each category


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syrin-labs/cli](https://github.com/syrin-labs/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
