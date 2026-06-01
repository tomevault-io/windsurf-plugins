---
trigger: always_on
description: This file provides guidance to CodeX when working with code in this repository.
---

# AGENTS.md

This file provides guidance to CodeX when working with code in this repository.

## Project Overview

Flock is a multi-provider AI agent desktop application with tool orchestration, sandbox execution, visual workflow, and browser/computer-use capabilities. It's a Rust workspace with a Tauri-based desktop UI (React + TypeScript + Mantine).

## Build & Development Commands

### Rust Backend
```bash
# Build the CLI
cargo build --release

# Run in single-shot mode
cargo run -- "your question"

# Run interactive REPL
cargo run

# Run with specific provider/profile
cargo run -- --profile deepseek "your question"

# Run tests (workspace-wide)
cargo test

# Run a single crate's tests
cargo test -p flock-core
cargo test -p flock-agent
cargo test -p flock-tools
cargo test -p flock-skills

# Run a specific test
cargo test -p flock-core test_name

# Lint
cargo clippy --workspace
```

### Tauri Desktop UI
```bash
cd flock-ui

# Install dependencies
npm install

# Development server
npm run dev
# or
npm run tauri dev

# Build for production
npm run build
# or
npm run tauri build

# Lint frontend
npm run lint
```

## Architecture

### Workspace Crates

| Crate | Purpose |
|-------|---------|
| `flock-core` | Core types, config, database, IPC interface, crypto, model factory |
| `flock-agent` | Agent engine, session management, tool execution, memory, graph orchestration, workflow engine |
| `flock-tools` | Tool registry, built-in tools (Read/Write/Edit/Bash/Grep/Glob), MCP integration, sandbox tools (CodeExecution, SandboxExec, Browser, ComputerUse) |
| `flock-skills` | Skill discovery, loading, frontmatter parsing, hooks, permissions, bundled skills |
| `flock-ui/src-tauri` | Tauri desktop app backend (commands, agent state, workspace management) |
| `workspace-hack` | cargo-hakari build optimization |

### Key Dependencies

- **LangGraph**: Rust reimplementation ([langgraph-rust](https://github.com/Onelevenvy/langgraph-rust)) for agent graph orchestration
- **LLM Providers**: OpenAI-compatible (default), Anthropic, AWS Bedrock, Google Vertex
- **Database**: SQLite via sqlx for sessions, conversations, providers, tools, MCP servers, workflows
- **Async Runtime**: Tokio
- **Sandbox**: Daytona cloud container runtime for isolated code execution
- **Browser Automation**: Playwright (Python, runs inside sandbox via CDP)

### Core Architecture Patterns

**Agent Engine** (`flock-agent/src/engine.rs`):
- `AgentEngine` orchestrates the LLM interaction loop
- Uses LangGraph `CompiledStateGraph` for execution flow
- Manages conversation history via SQLite checkpointer
- Handles tool approval, context compression, plan mode

**Tool System** (`flock-tools/src/lib.rs`):
- `Tool` trait defines the interface: `name()`, `description()`, `input_schema()`, `execute()`
- `ToolRegistry` manages all registered tools
- Built-in tools: Read, Write, Edit, Bash, Grep, Glob
- Sandbox tools: CodeExecution, SandboxExec, Browser, ComputerUse, RequestHumanAssistance
- Extensible via MCP servers and custom tool providers

**Workflow Engine** (`flock-agent/src/workflow_graph/`):
- Visual workflow builder with ReactFlow canvas
- 10 node types: start, llm, agent, classifier, ifelse, answer, code, human, plugin, parameter_extractor
- `build_workflow_graph()` compiles ReactFlow JSON into LangGraph `StateGraph`
- Streaming execution with human-in-the-loop interrupts

**Sandbox System** (`flock-tools/src/daytona/`):
- Cloud-based Daytona container runtime for isolated execution
- VNC desktop provisioning (Xvfb, fluxbox, x11vnc, websockify)
- Lifecycle management: create, destroy, set-public, snapshot
- Human takeover via VNC with approval flow

**Skills System** (`flock-skills/`):
- Markdown files with YAML frontmatter in `.flock/skills/`
- Supports `$ARGUMENTS` substitution, shell execution, conditional activation
- Two execution contexts: `inline` (same agent) and `fork` (sub-agent)
- Hot-reload via file watcher

**Memory System** (`flock-agent/src/memory/`):
- Four types: `user`, `feedback`, `project`, `reference`
- Stored as Markdown files with frontmatter in `<config>/flock/projects/<project>/memory/`
- `MEMORY.md` index file auto-loaded into system prompt

**IPC Interface** (`flock-core/src/ipc_interface/`):
- JSON-based protocol for host ↔ agent communication
- Events (Agent → Host): `ready`, `stream_start`, `text_delta`, `tool_request`, `tool_result`, etc.
- Commands (Host → Agent): `message`, `stop`, `tool_approve`, `tool_deny`, `set_mode`, `set_config`

**Configuration** (`flock-core/src/config/settings.rs`):
- Config priority: CLI args/env > project `flock.toml` > global config
- Supports profiles with inheritance (`extends`)
- Provider compatibility settings (`ProviderCompat`) for API differences

### Tauri Desktop UI

**Frontend Stack**:
- React 18 + TypeScript
- Mantine UI v8 (components, notifications)
- Zustand (state management)
- React Query (server state)
- react-markdown + react-syntax-highlighter (message rendering)
- i18next (internationalization)
- ReactFlow (workflow visual editor)

**State Stores** (`flock-ui/src/store/`):
- `agentStore`: Agent connection status, messages, pending approvals
- `uiStore`: Theme, sidebar state, active view, file tree

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Onelevenvy/flock](https://github.com/Onelevenvy/flock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
