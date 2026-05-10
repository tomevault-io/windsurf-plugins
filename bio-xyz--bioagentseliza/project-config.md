---
trigger: always_on
description: Guidelines for creating and maintaining Cursor rules to ensure consistency and effectiveness.
---

> You are an expert developer on the ElizaOS project. You write clear, correct, and maintainable code that adheres to the architectural principles and best practices outlined in this guide.

# ElizaOS Master Development Guide

## 1. Core Development Principles

- **Plan First**: Always begin with a thorough plan. For bugs, identify the root cause and all affected files. For features, create a complete implementation plan, analyzing risks and potential outcomes before writing code.
- **No Stubs**: Never commit incomplete code, stubs, or placeholder implementations. Write complete, working features.
- **Test-Driven**: Verify all changes with comprehensive tests. The system is complex, and models can be unpredictable. Write tests first when possible, and ensure all tests pass before considering a task complete.

## 2. Package & Architecture Overview

ElizaOS is a monorepo containing several key packages. Understanding their roles is crucial.

-   `packages/core`: `@elizaos/core` - The heart of the system, containing the `AgentRuntime` and all core type definitions (`Plugin`, `Action`, `Service`, etc.). **This package must not have dependencies on other packages in the monorepo.**
-   `packages/cli`: The command-line interface. This is the primary entry point for developers, used for running projects (`elizaos start`), managing agents (`elizaos agent`), and testing (`elizaos test`).
-   `packages/plugin-bootstrap`: Provides the default set of events, actions, and providers that give an agent its basic capabilities.
-   `packages/plugin-sql`: Provides the default database adapter for PGLite and PostgreSQL.

### High-Level Architecture
```mermaid
graph TD
    subgraph "Developer Interface"
        A[CLI (elizaos start)]
    end

    subgraph "Runtime"
        B[AgentServer]
        C[AgentRuntime]
        D[Database (PGLite/Postgres)]
        E[Plugins]
    end

    subgraph "Core Components (within Plugin)"
        F[Actions]
        G[Providers]
        H[Services]
        I[Events]
        J[Models]
    end

    A --> B;
    B --> C;
    C --> D;
    C --> E;
    E --> F & G & H & I & J;
```

---

## 3. The Plugin System

Plugins are the fundamental building blocks of the agent. Refer to [**Core Plugin Architecture**](mdc:.cursor/rules/elizaos_v2_api_plugins_core.mdc) for a deep dive.

- **`Plugin` Interface**: The manifest of capabilities (`name`, `description`, `init`, `dependencies`, `actions`, `providers`, `services`, etc.).
- **Lifecycle**: The `AgentRuntime` resolves dependencies, performs a topological sort, and then registers plugins in order, calling their `init` function and registering all components.
- **Services**: Long-running, stateful singleton classes for managing complex logic or connections (e.g., a database connection pool, a WebSocket client). Accessed via `runtime.getService('service-name')`.
- **Actions**: Define what an agent *can do*. A function that gets executed when the LLM decides to take an action.
- **Providers**: Supply contextual information (the agent's "senses") into the prompt before the LLM makes a decision.

---

## 4. CLI Usage & Project Management

The `elizaos` CLI is your primary tool for creating, running, and testing projects. Refer to [**CLI Project Management**](mdc:.cursor/rules/elizaos_v2_cli_project.mdc), [**Agent Management**](mdc:.cursor/rules/elizaos_v2_cli_agents.mdc), and [**Configuration Guide**](mdc:.cursor/rules/elizaos_v2_cli_config.mdc) for details.

### Key Commands
-   **`elizaos create`**: Interactively scaffolds a new project, plugin, or agent character file. It sets up the directory structure, `package.json`, and initial `.env` file.
-   **`elizaos start`**: Starts the `AgentServer`, loading the project or plugin from the current directory. This is the main command for running your agent locally.
-   **`elizaos test`**: Runs the test suite. It can execute both isolated unit tests (`vitest`) and end-to-end tests against a live, local runtime.
-   **`elizaos env`**: Provides commands (`list`, `edit-local`, `reset`) for safely managing your project-local `.env` file.

### Configuration
- **`.env` file**: Located at your project root, this is the primary place for all secrets and environment-specific configuration. It is loaded into `process.env` at runtime. **Never commit this file.**
- **`runtime.getSetting(key)`**: The correct way to access configuration from within your code. It provides a consistent interface to environment variables and character settings.

---

## 5. Testing

ElizaOS employs a two-pronged testing strategy: Unit tests for isolation and E2E tests for integration.

### Unit Testing
Refer to the [**Unit Testing Guide**](mdc:.cursor/rules/elizaos_v2_testing_unit.mdc).
- **Framework**: `vitest`.
- **Goal**: Test a single function or component in complete isolation.
- **Technique**: Use `vi.mock` to mock all external dependencies, especially the `IAgentRuntime`. Create mock factories to provile ###esting
Refer to the [**E2E Testing Guide**](mdc:.cursor/rules/2_testing_e2e.mdc).
- **Frameworlizaos test` test runner.
- **Gointegration and interaction of multiple components in a live environment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bio-xyz/BioAgentsEliza](https://github.com/bio-xyz/BioAgentsEliza) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
