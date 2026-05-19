---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This monorepo contains:
- **edda/** - Rust-based event-sourced AI agent orchestration system (primary focus)
- **klaudbiusz/** - Python wrapper around Claude Agent SDK using edda MCP (secondary focus)
- **agent/** - Legacy Python implementation (no longer maintained, mentioned for context only)

Active development happens in **edda/** with occasional work in **klaudbiusz/**.

## Edda - Event-Sourced Agent Orchestration (Rust)

### Architecture

Edda is a modular event-sourced AI agent system primarily exposed as an **MCP server** for existing agents (like Claude Code). Originally designed as a standalone agent, it was redesigned to focus on MCP integration. The standalone agent (edda_agent/edda_cli) is in early stage and postponed.

Core capabilities:
- **Event-Sourced Architecture**: Full event history with aggregate state reconstruction (edda_mq - stable)
- **Multi-Agent Coordination**: Link agents with bidirectional communication via `Link` trait
- **Pluggable LLM Support**: Provider-agnostic (Anthropic, Gemini via Rig)
- **Sandboxed Execution**: Dagger-based containerized tool execution
- **Type-Safe Event Handling**: Strongly-typed events, commands, responses

### Workspace Structure

| Crate | Purpose | Status |
|-------|---------|--------|
| **edda_mcp** | MCP server exposing scaffolding + Databricks tools | **Active (primary focus)** |
| **edda_sandbox** | Dagger-based containerized tool execution | **Active** |
| **edda_integrations** | External service integrations (Databricks, Google Sheets) | **Active** |
| **edda_templates** | Embedded application templates | **Active** |
| **edda_screenshot** | Browser automation for UI validation | **Active** |
| **edda_mq** | Event sourcing, aggregate management, persistence (SQLite/PostgreSQL) | Stable (infra for edda_agent) |
| **edda_agent** | Agent orchestration, event handling, coordination, toolbox | Early stage, postponed |
| **edda_cli** | CLI for agent execution | Early stage, postponed |

### Common Commands

```bash
cd edda

# Check all crates compile
cargo check

# Run examples (also serve as integration tests)
cargo run --example basic
cargo run --example planner_worker
cargo run --example multi_agent

# Build MCP server
cargo build --release --package edda_mcp

# Run MCP server (for development)
cargo run --manifest-path edda_mcp/Cargo.toml

# Run tests
cargo test

# Install MCP server locally
curl -LsSf https://raw.githubusercontent.com/appdotbuild/agent/refs/heads/main/edda/install.sh | sh
```

### Core Patterns

#### Agent Trait

All agents implement:
```rust
pub trait Agent: Default + Send + Sync + Clone {
    const TYPE: &'static str;
    type AgentCommand: Send;
    type AgentEvent: MQEvent;
    type AgentError: std::error::Error + Send + Sync + 'static;
    type Services: Send + Sync;

    async fn handle_tool_results(
        state: &AgentState<Self>,
        services: &Self::Services,
        incoming: Vec<ToolResult>,
    ) -> Result<Vec<Event<Self::AgentEvent>>, Self::AgentError>;

    async fn handle_command(
        state: &AgentState<Self>,
        cmd: Self::AgentCommand,
        services: &Self::Services,
    ) -> Result<Vec<Event<Self::AgentEvent>>, Self::AgentError>;

    fn apply_event(state: &mut AgentState<Self>, event: Event<Self::AgentEvent>);
}
```

#### Event Sourcing Pattern

Commands → Events → State:
1. Command received (e.g., `SendRequest`)
2. Handler validates and emits events
3. Events persisted to event store
4. State updated via `apply_event`
5. Listeners dispatch events to handlers

#### Multi-Agent Coordination

Use `Link` trait for agent-to-agent communication:
```rust
link_runtimes(&mut main_runtime, &mut specialist_runtime, CustomLink);
```

Example: Main agent delegates Databricks exploration to specialist agent with haiku model for cost optimization.

#### Tool Execution

All tools run in Dagger sandbox with isolated filesystem. Tools implement:
```rust
pub trait Tool: Send + Sync {
    fn name(&self) -> String;
    fn definition(&self) -> ToolDefinition;
    async fn call(&self, args: Value, sandbox: &mut impl Sandbox) -> Result<String>;
}
```

### Key Files

- **edda/docs/DESIGN.md** - Complete system design with diagrams
- **edda/CLAUDE.md** - Edda-specific development guidance
- **edda_agent/src/processor/agent.rs** - Agent trait and runtime
- **edda_agent/src/processor/link.rs** - Multi-agent coordination
- **edda_agent/src/toolbox/basic.rs** - Core tools (ReadFile, WriteFile, Bash, etc.)
- **edda_mq/src/aggregate.rs** - Event sourcing primitives
- **edda_mcp/src/main.rs** - MCP server implementation

### Environment Variables

```bash
# LLM providers
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=...

# Databricks (if using)
DATABRICKS_HOST=https://your-workspace.databricks.com
DATABRICKS_TOKEN=dapi...

# Logging
RUST_LOG=edda=debug,edda_agent=trace
```

### Adding New Components

**New Agent:**
1. Implement `Agent` trait with custom command/event types
2. Define `handle_tool_results` and `handle_command` methods
3. Implement `apply_event` for state reconstruction
4. Create `Runtime` with handlers (LLM, Tool, Log)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neondatabase/appdotbuild-agent](https://github.com/neondatabase/appdotbuild-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
