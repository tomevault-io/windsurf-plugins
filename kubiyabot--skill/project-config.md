---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Skill is a universal skill runtime for AI agents that executes tools in a sandboxed WASM environment. It provides both CLI commands and MCP (Model Context Protocol) server capabilities for AI agent integration.

## Build & Development Commands

```bash
# Setup (one-time)
rustup target add wasm32-wasip1  # Required for WASM skill compilation

# Build
cargo build                    # Debug build
cargo build --release          # Release build (optimized for size)
cargo build --profile release-fast  # Fast compile with speed optimization

# Test
cargo test --workspace         # Run all tests
cargo test -p skill-cli        # Test specific crate
cargo test -p skill-cli test_name  # Run single test by name
cargo test -p skill-cli --lib -- claude_bridge  # Test claude_bridge module
cargo test -p skill-runtime -- --test-threads=1  # Serial execution for flaky tests
./tests/mcp_integration_tests.sh  # MCP protocol tests
./tests/run-all-tests.sh       # Full test suite

# Snapshot testing (uses insta crate)
cargo insta test               # Run snapshot tests
cargo insta review             # Review and accept new snapshots

# Benchmarks
cargo bench -p skill-cli       # Run benchmarks (claude_bridge_bench)

# Lint & Format
cargo fmt --all                # Format code
cargo clippy --all-targets -- -D warnings  # Lint

# Coverage
cargo tarpaulin --workspace --out Html  # Generate HTML coverage report

# Install locally
cargo install --path crates/skill-cli

# Documentation site (VitePress)
cd docs-site && npm install    # One-time setup
cd docs-site && npm run dev    # Local dev server
cd docs-site && npm run build  # Production build
```

## Architecture

### Crate Structure

```
crates/
├── skill-cli/        # Main binary - CLI commands, Claude bridge, auth
├── skill-runtime/    # Core execution engine - WASM, Docker, native runners
├── skill-mcp/        # MCP server implementation (stdio transport)
├── skill-http/       # HTTP streaming server for web clients
├── skill-web/        # Yew-based WebAssembly UI
└── skill-context/    # Shared types and context
```

### Key Components

**skill-runtime** is the core engine providing:
- `SkillEngine` - Orchestrates execution and search
- `SkillManifest` - Declarative skill configuration (`.skill-engine.toml`)
- `SkillExecutor` - WASM Component Model execution via Wasmtime
- `DockerRuntime` - Containerized skill execution
- `SearchPipeline` - RAG-powered semantic search (FastEmbed, optional Qdrant)
- Native execution for SKILL.md-based skills (kubectl, git, etc.)

**skill-cli** commands in `crates/skill-cli/src/commands/`:
- `install.rs` - Install from local, HTTP, Git sources
- `run.rs` - Execute skill tools with arguments
- `serve.rs` - Start MCP server (stdio or HTTP)
- `find.rs` - Semantic search for tools
- `list.rs` - List installed skills
- `info.rs` - Show skill details
- `remove.rs` - Uninstall skills
- `config.rs` - Configure skill credentials
- `claude.rs` - Claude Code integration setup (`skill claude setup/status/remove`)
- `claude_bridge/` - Generates Claude Code-compatible skill definitions from SKILL.md files

**skill-mcp** exposes three MCP tools:
- `execute` - Run any skill tool
- `list_skills` - Paginated skill listing
- `search_skills` - Semantic search

### Runtime Feature Flags

The runtime has optional features that enable additional capabilities:
```bash
cargo build -p skill-runtime --features hybrid-search,reranker,context-compression,qdrant
```

- `hybrid-search` - BM25 + vector fusion (tantivy)
- `reranker` - Cross-encoder reranking (fastembed)
- `context-compression` - Token-aware compression (tiktoken-rs)
- `qdrant` - Production vector database
- `ai-ingestion` - LLM providers for example generation
- `job-queue` - Async job scheduling (apalis)

## Key Patterns

### SKILL.md Format

Native skills use SKILL.md markdown files with YAML frontmatter:
```markdown
---
name: kubernetes
description: Kubernetes management
allowed-tools: Bash
---

### get
Get Kubernetes resources

**Parameters:**
- `resource` (required, string): Resource type
- `namespace` (optional, string): Namespace
```

Parameter parsing rules in `skill-runtime/src/skill_md.rs`.

### Manifest Configuration

`.skill-engine.toml` declares skills and instances:
```toml
[skills.kubernetes]
source = "./examples/native-skills/kubernetes-skill"

[skills.kubernetes.instances.prod]
config = { context = "prod-cluster" }
```

Manifest loading in `skill-cli/src/commands/manifest.rs`.

### MCP Protocol

MCP server uses stdio for Claude Code integration. **Critical: all logs must go to stderr** (stdout is reserved for JSON-RPC). Use `tracing` macros which are configured to write to stderr. Entry point in `skill-mcp/src/server.rs`.

## Testing

- Unit tests: Same file with `#[cfg(test)]` modules
- Integration tests: `crates/*/tests/`
- Shell tests: `tests/` directory (MCP, security, e2e)
- Snapshot tests: Uses `insta` crate for YAML snapshots

Run specific test:
```bash
cargo test -p skill-cli test_name
cargo test -p skill-runtime -- --test-threads=1  # Serial execution
```

## Execution Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubiyabot/skill](https://github.com/kubiyabot/skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
