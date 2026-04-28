---
trigger: always_on
description: > This file provides AI coding agents with the context needed to work effectively on this project.
---

# AGENTS.md — Cowork Forge

> This file provides AI coding agents with the context needed to work effectively on this project.
> For project knowledge (architecture, decisions, issues), see [`.ai-context/SKILL.md`](.ai-context/SKILL.md).

---

## Project Overview

**Cowork Forge** is an AI-native multi-agent software development platform. It orchestrates specialized AI agents (Product Manager, Architect, Project Manager, Engineer) through a 7-stage pipeline to transform ideas into production-ready software.

| Aspect | Detail |
|--------|--------|
| Language | Rust (edition 2024) |
| Agent Framework | adk-rust 0.5.0 |
| GUI | Tauri + React 18 + Ant Design |
| Architecture | Hexagonal + DDD |
| License | MIT |

### Workspace Structure

```
crates/
├── cowork-core/         # Domain logic, pipeline, tools, agents (MAIN crate)
│   └── src/
│       ├── pipeline/    # 7-stage orchestration & stage executor
│       ├── domain/      # Project, Iteration, Memory aggregates
│       ├── tools/       # 40+ ADK tools + MCP integration
│       ├── agents/      # Agent wrappers (iterative, PM, legacy analyzer)
│       ├── interaction/ # InteractiveBackend trait (CLI/GUI abstraction)
│       ├── acp/         # Agent Client Protocol for external agents
│       ├── config_definition/  # Data-driven config (agents, stages, flows)
│       ├── instructions/       # Agent prompt library
│       ├── skills/      # agentskills.io standard skill system
│       ├── integration/ # Hook manager for external integrations
│       └── persistence/ # JSON-based storage
├── cowork-cli/          # CLI adapter (clap + dialoguer)
└── cowork-gui/          # Tauri + React GUI
    ├── src-tauri/       # Rust backend (Tauri commands + events)
    └── src/             # React frontend (TypeScript + Ant Design)
```

---

## Dev Environment Setup

### Prerequisites

- **Rust** (edition 2024, stable toolchain)
- **Node.js** (for GUI frontend build)
- **LLM API Key** (OpenAI-compatible endpoint)

### Build

```bash
# Build entire workspace
cargo build

# Release build
cargo build --release

# Build GUI only (installs frontend deps automatically)
cd crates/cowork-gui && cargo tauri dev
```

### Run

```bash
# CLI
cargo run --package cowork-cli -- <command>

# GUI (development mode)
cd crates/cowork-gui && cargo tauri dev
```

### Configuration

Config file location:

| Platform | Path |
|----------|------|
| Windows | `%APPDATA%\CoworkCreative\config.toml` |
| macOS | `~/Library/Application Support/CoworkCreative/config.toml` |
| Linux | `~/.config/CoworkCreative/config.toml` |

User-facing config directory:

| Platform | Path |
|----------|------|
| Windows | `%APPDATA%\com.cowork-forge.app\config\` |
| macOS | `~/Library/Application Support/com.cowork-forge.app/config/` |
| Linux | `~/.config/com.cowork-forge.app/config/` |

---

## Build and Test Commands

```bash
# Run all tests
cargo test

# Test a specific crate
cargo test -p cowork-core

# Test a specific module
cargo test -p cowork-core pipeline

# Run with all features
cargo test --all-features

# Check compilation without building
cargo check

# Lint (if clippy configured)
cargo clippy
```

### GUI Frontend

```bash
cd crates/cowork-gui

# Install dependencies
npm install    # or: bun install

# Build frontend only
npm run build

# Development server
npm run dev
```

---

## Code Style and Conventions

### Rust

- **Error handling**: Always use `anyhow::Result`. Never use `unwrap()` in production code.
- **Async traits**: Use `async_trait` for async trait methods.
- **Naming**: `snake_case` for functions/variables, `PascalCase` for types/traits.
- **Architecture**: Follow hexagonal architecture — domain logic has zero external dependencies. Infrastructure adapters implement domain ports.
- **Trait-based abstraction**: `InteractiveBackend` is the key port for CLI/GUI abstraction. All user interaction flows through this trait.
- **Serialization**: `serde` with derive macros for all domain entities.
- **Async runtime**: Tokio with `features = ["full"]`.

### TypeScript / React (GUI)

- Component-based architecture with Ant Design.
- Tauri commands for request-response, events for streaming.
- State management via React hooks.

### Key Patterns

| Pattern | Where | Purpose |
|---------|-------|---------|
| Actor-Critic | PRD, Design, Plan, Coding stages | Iterative self-refinement |
| Strategy | Stage trait implementations | Pluggable stage behavior |
| Template Method | Pipeline execution flow | Fixed stage sequence with hooks |
| Repository | Persistence stores | Abstract data access |
| Decorator | LLM rate limiting | Transparent cross-cutting concern |

---

## Key Files

When working on specific areas, start from these files:

| Area | Primary File | Related |
|------|-------------|---------|
| Pipeline execution | `crates/cowork-core/src/pipeline/executor/mod.rs` | `stage_executor.rs`, `knowledge.rs` |
| Stage implementations | `crates/cowork-core/src/pipeline/stages/*.rs` | 7 stage files: idea, prd, design, plan, coding, check, delivery |
| Tool implementations | `crates/cowork-core/src/tools/mod.rs` | `file_tools.rs`, `data_tools.rs`, `hitl_tools.rs`, `pm_tools.rs`, etc. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sopaco/cowork-forge](https://github.com/sopaco/cowork-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
