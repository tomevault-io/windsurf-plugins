---
trigger: always_on
description: This project adheres to the most rigorous standards of Idiomatic Rust. We do not compromise on memory safety, ownership patterns, or performance for convenience.
---

# CLAUDE.md — specre

## Rust Philosophy & Strictness
This project adheres to the most rigorous standards of Idiomatic Rust. We do not compromise on memory safety, ownership patterns, or performance for convenience.

- **The Compiler is King:** Treat every compiler warning as a fatal error. If the editor turns red, it is not a "suggestion" to ignore; it is a sign that the current implementation is flawed and unidiomatic.
- **Embrace the Friction:** Do not seek "easy" workarounds (like excessive `.clone()`, `Box<dyn T>`, or `unwrap()`). The friction provided by the borrow checker is a gift that ensures long-term stability.
- **Clippy is our Moral Compass:** This project uses `#![deny(clippy::pedantic, clippy::nursery, clippy::all)]`. Your goal is to produce code that passes these checks without suppressions (e.g., no `#[allow(...)]` without a profound architectural justification).
- **Atonement for Errors:** When code fails to compile or violates Clippy rules, view it as a failure of logic. Apologize only through superior, idiomatic refactoring. Never suggest that the user "ignore" or "downgrade" these strict settings.

リファクタやコード品質改善タスクにあたっては、[docs/guides/RUST-CONVENTIONS.md](docs/guides/RUST-CONVENTIONS.md) の具体的なルールを参照せよ。

## What is specre?

Atomic, living specification cards for AI-agent-friendly development. A Rust CLI toolkit for Spec-Driven Development (SDD). Each specre is a single Markdown file describing exactly one behavior, with YAML front-matter for lifecycle tracking and bidirectional traceability.

**Core philosophy:** One specre card = one behavior. Sized for a single LLM context window.

## Project Structure

```
specre/
├── Cargo.toml               # Rust package manifest (v0.2.6, edition 2024)
├── specre.toml              # Project config (created by `specre init`)
├── src/
│   ├── main.rs              # Entry point (clap CLI routing)
│   ├── cli.rs               # CLI argument definitions (clap derive)
│   ├── config.rs            # specre.toml parser (serde + toml)
│   ├── error.rs             # Error types and contextual diagnostics
│   ├── status.rs            # Status enum and lifecycle logic
│   ├── template.rs          # specre card template generation
│   ├── ulid.rs              # ULID generation wrapper
│   └── commands/
│       ├── mod.rs
│       ├── init.rs          # `specre init` implementation
│       ├── new.rs           # `specre new` implementation
│       ├── index.rs         # `specre index` implementation
│       ├── status.rs        # `specre status` implementation
│       ├── trace.rs         # `specre trace` implementation
│       ├── orphans.rs       # `specre orphans` implementation
│       ├── tag.rs           # `specre tag` implementation
│       ├── search.rs        # `specre search` implementation
│       ├── coverage.rs      # `specre coverage` implementation
│       ├── health_check.rs  # `specre health-check` implementation
│       └── mcp.rs           # `specre mcp` MCP server implementation
├── tests/                   # Integration tests (assert_cmd)
│   ├── cli_init.rs          # ... and cli_new, cli_index, cli_status,
│   ├── cli_trace.rs         #     cli_orphans, cli_tag, cli_search,
│   ├── cli_coverage.rs      #     cli_health_check, cli_dispatch,
│   ├── cli_json_output.rs   #     cli_json_output
│   └── mcp/                 # MCP server integration tests
└── docs/specres/            # specre cards for this project itself
    ├── cli/
    └── mcp/
```

## Tech Stack

- **Language:** Rust (2024 edition)
- **CLI framework:** clap v4 (derive)
- **MCP server:** rmcp (stdio transport), tokio, schemars, tracing
- **Date handling:** chrono
- **Test framework:** assert_cmd + assert_fs + predicates (integration tests)
- **Build:** `cargo build` / `cargo test`

## CLI Commands

| Command | Status | Description |
|---------|--------|-------------|
| `specre init` | Implemented | Initialize project (creates `specre.toml` and specre directory) |
| `specre new <dir> --name <name>` | Implemented | Scaffold a new specre card with auto-generated ULID |
| `specre index` | Implemented | Generate `index.json` and per-domain `_INDEX.md` |
| `specre status` | Implemented | Report specre counts by status, flag stale `last_verified` |
| `specre trace` | Implemented | Bidirectional traceability lookup by ULID or file path |
| `specre orphans` | Implemented | Detect unlinked specres or dangling markers |
| `specre tag` | Implemented | Insert `@specre` markers into source files |
| `specre search` | Implemented | Full-text + filter search with JSON output |
| `specre coverage` | Implemented | Report percentage of source files covered by `@specre` tags |
| `specre health-check` | Implemented | Comprehensive health check for AI agent preflight |
| `specre mcp` | Implemented | Start MCP server (stdio transport) |
| `specre drift` | Not yet | Detect drift between spec and implementation |
| `specre ci` | Not yet | CI integration (non-zero exit on drift/orphans) |

## Development Workflow

### Implementing a new specre CLI command


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yoshiakist) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
