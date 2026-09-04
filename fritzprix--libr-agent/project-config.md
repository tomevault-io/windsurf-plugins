---
trigger: always_on
description: **LibrAgent: A High-Freedom AI Agent Platform - Infinitely Expandable with MCP!**
---

# 🚀 LibrAgent Project Guidelines

## Project Overview

**LibrAgent: A High-Freedom AI Agent Platform - Infinitely Expandable with MCP!**

LibrAgent is a next-generation desktop AI agent platform that combines the lightness of Tauri with the intuitiveness of React. Users can automate all daily tasks by giving AI agents their own unique personalities and abilities.

## Key Architecture Patterns

**Agent V2 Architecture (Session-Isolated):**

- **Per-Session Tool Instances**: Each agent session gets isolated `MCPServiceProxy` with dedicated builtin server instances
- **Session-Specific MCP Managers**: Separate `HttpSessionManager` and `SessionMCPManager` per session
- **Context Registry System**: Dynamic context providers (time/location, skills) inject state into system prompts
- **Rust-Orchestrated Workflows**: Think-Act-Observe loop managed entirely in Rust backend (`AgentSessionManager`)

**MCP Integration Architecture:**

- **External MCP Servers**: Stdio/HTTP protocol via `rmcp` library, managed by session-isolated managers
- **Builtin MCP Servers**: Native Rust implementations via `BuiltinMCPServer` trait
  - Planning, Knowledge, Browser, Workspace, Content Store, etc.
  - Session-isolated instances with dedicated state
- **Unified Tool Discovery**: `MCPServiceProxy` routes calls to builtin or external servers transparently

**Feature-Based Organization:**

- Each feature in `src/features/` contains components, hooks, and README documentation
- Compound component patterns (e.g., `Chat.Header`, `Chat.Messages`, `Chat.Input`)
- React Context providers for state sharing (`ChatProvider`, `AgentSessionProvider`, `AgentChatProvider`)

**Service Layer Pattern:**

- `src/lib/backend/` contains Tauri command wrappers with centralized `safeInvoke()` utility
- Centralized logging via `getLogger('ComponentName')` instead of console methods
- All API communication through typed service modules with error handling

**Key Features:**

- **AI Agent Management**: Session-isolated agents with independent tool state and context
- **LLM Provider Support**: 8 providers, 50+ models including reasoning models (o3, DeepSeek R1)
- **Built-in Tool Ecosystem**: Planning, Knowledge, Browser, Workspace, Code Execution
- **MCP Integration**: Session-isolated stdio/HTTP protocol with security validation

## Technology Stack

**Core Framework:**

- PNPM (Package Manager)
- Tauri 2.x (Latest cross-platform desktop framework)
- React 18.3 (Modern UI with concurrent features)
- TypeScript 5.6 (Advanced type safety)
- `rmcp` 0.8.x (Rust-based Model Context Protocol client; see `src-tauri/Cargo.toml`)

**Frontend Technologies:**

- Tailwind CSS 4.x (Latest utility-first styling)
- Radix UI (Accessible component primitives)
- SeaORM (Database ORM for SQLite, used via Rust backend)
- React Context (State management with providers)
- Vite 6.x (Fast development and build tool)

**Backend Technologies:**

- Rust (High-performance native operations)
- Tokio 1.49+ (Async runtime for concurrent operations)
- SeaORM (Type-safe database ORM for SQLite)
- rmcp 0.8.1+ (MCP client library with stdio/HTTP transport)
- reqwest 0.12+ (HTTP client for MCP servers and browser automation)

## Development Scripts & Workflow

LibrAgent provides several useful scripts for development and code quality:

- `pnpm dev` – Start the Vite development server
- `pnpm tauri dev` – Start the Tauri desktop app with hot reload
- `pnpm build` – Build the frontend for production
- `pnpm lint` – Run ESLint checks for code quality
- `pnpm format` – Format code using Prettier
- `pnpm rust:fmt` – Check Rust code formatting
- `pnpm rust:clippy` – Run Rust linter
- `pnpm dead-code` – Find unused code with unimported
- `pnpm refactor:validate` – **Complete validation pipeline:**  
  Runs lint, format, Rust validation, build, and dead-code checks.  
  **Always run this after any development or refactoring work to ensure code quality and build integrity.**

**Workflow Recommendation:**  
After making any code changes, always run:

```sh
pnpm refactor:validate
```

**Execution rule:** Prefer repository `pnpm` scripts for validation and Rust workflow commands. Do **not** default to invoking `cargo` directly for routine validation in this repo, because the wrapper scripts encode the expected environment and direct `cargo run`/ad-hoc cargo execution can crash or diverge from repository behavior.

**Review delegation rule:** Review-agent/sub-agent review is allowed, but only when the prompt is explicit and tightly scoped. When delegating review, name the exact files, subsystem, risk area, and review goal (for example: correctness regression, state handling, contract mismatch, or data-loss risk), and instruct the agent to report only concrete high-signal issues. Do **not** throw a vague “review the PR” prompt at a review agent, because that invites noise, busywork, and dumb detours.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fritzprix/libr-agent](https://github.com/fritzprix/libr-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
