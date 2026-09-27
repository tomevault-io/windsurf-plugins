---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Codify is a configuration-as-code CLI tool that brings Infrastructure-as-Code principles to local development environments. It allows developers to declaratively define their development setup (packages, tools, system settings) in configuration files and apply them in a reproducible way. Think "Terraform for your local machine."

## Development Commands

### Building
```bash
npm run build              # Build TypeScript to dist/
npm run lint               # Type-check with tsc
```

### Testing
```bash
npm test                   # Run all tests with Vitest
npm test -- path/to/test   # Run specific test file
npm run posttest           # Runs lint after tests
```

### Running Locally
```bash
./bin/dev.js <command>     # Run CLI in development mode
./bin/dev.js apply         # Example: run apply command
```

### Test Command (VM Testing)
The `test` command spins up a Tart VM to test Codify configs in isolation:
```bash
./bin/dev.js test --vm-os darwin   # Test on macOS VM
./bin/dev.js test --vm-os linux    # Test on Linux VM
```

## High-Level Architecture

### Core Architectural Patterns

1. **Command-Orchestrator Pattern**: Commands (`src/commands/`) are thin oclif wrappers. Orchestrators (`src/orchestrators/`) contain all business logic and workflow coordination. This separation enables reusability.

2. **Multi-Process Plugin System**: The most unique architectural decision is running plugins as separate Node.js child processes communicating via IPC:
   - **Why**: Isolation (crashes don't crash CLI), security (parent controls sudo), flexibility
   - **Plugin Process** (`src/plugins/plugin-process.ts`): Spawns plugins using `fork()`
   - **IPC Protocol** (`src/plugins/plugin-message.ts`): Type-safe message passing
   - **Security**: Plugins run isolated; parent process controls all sudo operations
   - When plugins need sudo, they send `COMMAND_REQUEST` events back to parent

3. **Event-Driven Architecture**: Central event bus (`src/events/context.ts`) using EventEmitter:
   - Tracks process/subprocess lifecycle (PLAN, APPLY, INITIALIZE_PLUGINS, etc.)
   - Enables plugin-to-CLI communication (sudo prompts, login credentials, etc.)
   - Powers progress tracking for UI

4. **Reporter Pattern**: Abstract `Reporter` interface with multiple implementations selected via `--output` flag:
   - `DefaultReporter`: Rich Ink-based TUI with React components
   - `PlainReporter`: Simple text output
   - `JsonReporter`: Machine-readable JSON
   - `DebugReporter`: Verbose logging
   - `StubReporter`: No-op for testing

5. **Resource Lifecycle State Machine**:
   ```
   Parse Config → Validate → Resolve Dependencies → Plan → Apply
   ```
   - **ResourceConfig**: Desired state from config file
   - **Plan**: Computed difference between desired and current state
   - **ResourcePlan**: Per-resource operations (CREATE, UPDATE, DELETE, NOOP)
   - **Project**: Container with dependency graph

6. **Dependency Resolution**:
   - Explicit: `dependsOn` field in config
   - Implicit: Extracted from parameter references (e.g., `${other-resource.param}`)
   - Plugin-level: Plugins declare type dependencies (e.g., xcode-tools on macOS)
   - Topological sort ensures correct evaluation order (`src/utils/dependency-graph-resolver.ts`)

### Key Directory Structure

- **`/src/orchestrators/`**: Business logic layer - each file implements one CLI command's workflow
  - `plan.ts`: Parse → Validate → Resolve deps → Generate plan
  - `apply.ts`: Execute plan after user confirmation
  - `import.ts`: Import existing resources into config
  - `test.ts`: VM-based testing with live config sync via file watcher

- **`/src/plugins/`**: Plugin infrastructure
  - `plugin-manager.ts`: Registry routing operations to plugins
  - `plugin-process.ts`: Child process lifecycle and IPC
  - `plugin.ts`: High-level plugin API

- **`/src/entities/`**: Domain models with rich behavior
  - `Project`: Container with dependency resolution
  - `ResourceConfig`: Mutable config with dependency tracking
  - `Plan`: Immutable plan with sorting/filtering

- **`/src/parser/`**: Multi-format config parsing (JSON, JSONC, JSON5, YAML)
  - All parsers maintain source maps for error messages
  - Cloud parser fetches from Dashboard API via UUID

- **`/src/ui/`**: User interface layer
  - `/reporters/`: Output strategy implementations
  - `/components/`: React components for Ink TUI
  - `/store/`: Jotai state management for UI

- **`/src/connect/`**: Dashboard integration
  - WebSocket server for persistent connection
  - OAuth flow handling
  - JWT credential management

- **`/src/generators/`**: Config file writers
  - Computes diffs for updating existing configs
  - Writes to local files or cloud (via Dashboard API)

### Important Data Flows

**Apply Command Flow:**
```
ApplyOrchestrator.run()
  → PlanOrchestrator.run()
    → PluginInitOrchestrator.run()
      → Parse configs → Project
      → PluginManager.initialize() → ResourceDefinitions
    → Project.resolveDependencies()
    → PluginManager.plan() → Plan
  → Reporter.promptConfirmation()
  → PluginManager.apply()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codifycli/codify](https://github.com/codifycli/codify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
