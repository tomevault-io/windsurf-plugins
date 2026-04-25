---
trigger: always_on
description: **Last Updated**: 2025-11-23
---

# GitHub Copilot Instructions for MCP Execution

**Last Updated**: 2025-11-23

This document provides comprehensive guidelines for GitHub Copilot to generate high-quality code for the MCP Execution project, synthesizing expertise from architectural design, development practices, testing strategies, performance optimization, security, code review, and CI/CD.

---

## Table of Contents

1. [Project Context](#project-context)
2. [Core Principles](#core-principles)
3. [Code Generation Guidelines](#code-generation-guidelines)
4. [Architecture & Design Patterns](#architecture--design-patterns)
5. [Error Handling](#error-handling)
6. [Testing Requirements](#testing-requirements)
7. [Performance Optimization](#performance-optimization)
8. [Security Requirements](#security-requirements)
9. [Documentation Standards](#documentation-standards)
10. [CI/CD Integration](#cicd-integration)
11. [Code Review Checklist](#code-review-checklist)

---

## Project Context

### Project Overview

**MCP Execution** is a Rust workspace implementing the Code Execution pattern for Model Context Protocol (MCP) using WebAssembly for secure, isolated code execution.

**Key Metrics**:
- 90%+ token reduction target
- <50ms execution overhead
- Zero sandbox escapes
- 314+ tests (100% passing)
- Cross-platform support (Linux, macOS, Windows)

### Workspace Structure

```
mcp-execution/
├── crates/
│   ├── mcp-execution-core/          - Foundation: types, traits, errors
│   ├── mcp-execution-introspector/  - MCP server analysis (rmcp v0.9)
│   ├── mcp-execution-codegen/       - Code generation (wasm/skills features)
│   ├── mcp-bridge/        - WASM ↔ MCP proxy (rmcp client)
│   ├── mcp-wasm-runtime/  - Wasmtime sandbox (v39.0)
│   ├── mcp-execution-files/           - Virtual filesystem
│   ├── mcp-cli/           - CLI application
│   ├── mcp-execution-skill-store/   - Skill management
│   ├── mcp-execution-skill-generator/ - Skill generation
│   └── mcp-examples/      - Usage examples
└── .local/                - Detailed documentation (not in git)
```

### Technology Stack

- **Rust Edition**: 2024 (MSRV: 1.89)
- **MCP SDK**: rmcp v0.9 (official Rust SDK)
- **WASM Runtime**: Wasmtime v39.0
- **Async Runtime**: Tokio v1.48
- **Testing**: cargo-nextest, criterion
- **CI/CD**: GitHub Actions with cargo-deny

---

## Core Principles

### 1. Microsoft Rust Guidelines Compliance

**CRITICAL**: All code MUST follow [Microsoft Rust Guidelines](https://microsoft.github.io/rust-guidelines/agents/all.txt).

**Key Requirements**:

- **Strong Types Over Primitives**: Use `ServerId`, `ToolName`, `SessionId`, etc., not `String` or `u64`
- **Error Handling**: `thiserror` for library crates, `anyhow` only for CLI (`mcp-cli`)
- **Public Types**: Must implement `Send + Sync + Debug`
- **Documentation**: Every public item needs doc comments with examples
- **Safety**: `unsafe` only for FFI or novel abstractions; must include `SAFETY` comments

**Example - Strong Types**:
```rust
// ✅ GOOD: Type-safe IDs
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct ServerId(u64);

impl ServerId {
    pub fn new(id: u64) -> Self {
        Self(id)
    }

    pub fn as_u64(&self) -> u64 {
        self.0
    }
}

// ❌ BAD: Primitive types
pub fn connect_server(server_id: u64) -> Result<Connection> { ... }
```

### 2. Official MCP SDK Usage

**CRITICAL**: Use **rmcp v0.9** for all MCP communication.

```rust
// ✅ GOOD: Use rmcp client
use rmcp::client::TokioChildProcess;

let transport = TokioChildProcess::new("github")?;
let client = rmcp::client::Client::new(transport);

// ❌ BAD: Custom MCP protocol implementation
// NEVER implement custom JSON-RPC for MCP
```

### 3. Security First

- WASM sandbox with memory/CPU limits
- No network access from WASM (only via MCP Bridge)
- WASI preopened directories only
- Rate limiting on all tool calls
- No hardcoded secrets
- Input validation on all external data

### 4. Naming Conventions

- **Crates**: `mcp-{feature}` (kebab-case)
- **Files & modules**: `snake_case`
- **Types & traits**: `PascalCase`
- **Functions & variables**: `snake_case`
- **Constants**: `SCREAMING_SNAKE_CASE`

### 5. Version Control & File Management

**CRITICAL**: Working and intermediate files must NEVER be committed to the repository.

**Working/Intermediate Files Location**:
- All temporary, draft, and working documents belong in `.local/` directory
- `.local/` is excluded from version control (in `.gitignore`)
- Use `.local/` for implementation plans, status reports, research notes, etc.

**What to commit**:
- ✅ Source code (`src/`, `tests/`, `benches/`)
- ✅ Configuration files (`Cargo.toml`, `.github/`, `deny.toml`)
- ✅ Documentation (`README.md`, `docs/`, ADRs)
- ✅ Project metadata (`LICENSE`, `CHANGELOG.md`)

**What NOT to commit**:
- ❌ Working documents and drafts (→ use `.local/`)
- ❌ Build artifacts (`target/`, `*.wasm`)
- ❌ IDE files (`.vscode/`, `.idea/`)
- ❌ Temporary files (`*.tmp`, `*.swp`, `*.bak`)
- ❌ Sensitive data (`.env`, credentials)
- ❌ Generated reports and analysis outputs (→ use `.local/`)

**Example**:
```bash
# ✅ GOOD: Working documents in .local/
.local/
├── implementation-plan.md
├── performance-analysis.md
├── security-audit-results.md
└── meeting-notes.md

# ❌ BAD: Don't commit these to root or docs/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bug-ops/mcp-execution](https://github.com/bug-ops/mcp-execution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
