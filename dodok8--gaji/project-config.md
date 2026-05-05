---
trigger: always_on
description: **gaji** is a Rust CLI tool that enables type-safe GitHub Actions workflow authoring in TypeScript. Users write workflows in TypeScript with full type safety and autocomplete, and gaji compiles them to YAML files that GitHub Actions understands.
---

# CLAUDE.md

## Project Overview

**gaji** is a Rust CLI tool that enables type-safe GitHub Actions workflow authoring in TypeScript. Users write workflows in TypeScript with full type safety and autocomplete, and gaji compiles them to YAML files that GitHub Actions understands.

- **Repository**: https://github.com/dodok8/gaji
- **Language**: Rust (edition 2021)
- **License**: MIT
- **Version**: defined in `Cargo.toml`

## Quick Reference Commands

```bash
# Build the project
cargo build

# Build release binary (optimized for size)
cargo build --release

# Run all tests
cargo test --all-features

# Lint check
cargo clippy --all-targets --all-features -- -D warnings

# Format check
cargo fmt --all --check

# Format code
cargo fmt --all
```

## Writing Docs

Before Writing docs, See <https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing> and avoid these patterns.

## Project Structure

```
src/
├── main.rs          # CLI entry point, command routing via clap
├── lib.rs           # Public module exports
├── cli.rs           # CLI command/argument definitions (clap derive)
├── config.rs        # Configuration loading (gaji.config.ts / gaji.config.local.ts)
├── builder.rs       # TypeScript → YAML workflow compilation pipeline
├── executor.rs      # JavaScript execution via QuickJS runtime
├── cache.rs         # Action metadata caching (.gaji-cache.json)
├── fetcher.rs       # GitHub API client for fetching action.yml files
├── watcher.rs       # File system watcher for dev mode (notify crate)
├── parser/
│   ├── mod.rs       # Parser module interface
│   ├── ast.rs       # AST visitor for extracting action references
│   └── extractor.rs # Action reference extraction from TypeScript code
├── generator/
│   ├── mod.rs       # Type generation orchestration
│   ├── types.rs     # TypeScript type definition (.d.ts) generation
│   └── templates.rs # Hardcoded type templates (base types, runtime)
└── init/
    ├── mod.rs       # Project initialization & state detection
    ├── interactive.rs  # Interactive init mode (dialoguer)
    ├── migration.rs    # YAML → TypeScript migration
    └── templates.rs    # Template files for new projects

tests/
└── integration.rs   # End-to-end builder→executor→YAML pipeline tests

workflows/           # gaji's own CI workflows (self-dogfooding)
├── audit.ts
├── ci.ts
├── release-plz.ts
├── release.ts
├── update-workflows.ts
└── vitepress.ts

npm/                 # NPM distribution wrapper
├── gaji/            # Main npm package (postinstall downloads binary)
└── platform-*/      # Platform-specific binary packages
```

## Architecture

The core pipeline is: **TypeScript → Parse → Execute → YAML**

1. **Parser** (`parser/`): Uses oxc to parse TypeScript and extract `getAction("owner/repo@version")` calls via AST visitor pattern
2. **Fetcher** (`fetcher.rs`): Downloads `action.yml` from GitHub for referenced actions
3. **Generator** (`generator/`): Generates TypeScript type definitions (`.d.ts`) from action metadata
4. **Executor** (`executor.rs`): Strips TypeScript types with oxc, bundles with runtime JS, executes in QuickJS
5. **Builder** (`builder.rs`): Orchestrates the full pipeline, converts JSON output to YAML, writes workflow files

## Key Design Patterns

- **Builder pattern**: `WorkflowBuilder`, `Job`, `Workflow`, `WorkflowCall`, `ActionRef`, `Action`, `NodeAction`, `DockerAction`
- **Callback builder pattern**: `StepBuilder` accumulates step context (`_ctx`), `JobBuilder` accumulates job output context — callbacks receive previous outputs for type-safe chaining
- **Visitor pattern**: `ActionRefExtractor` traverses oxc AST nodes
- **Error handling**: `anyhow::Result<T>` with `?` propagation throughout; `thiserror` for typed errors
- **Async**: Tokio runtime for all I/O-bound operations (HTTP, filesystem)
- **Configuration hierarchy**: env vars > `gaji.config.local.ts` > `gaji.config.ts` > defaults

## Code Conventions

- **Modules**: `snake_case` (e.g., `type_generator`)
- **Types/Structs**: `PascalCase` (e.g., `WorkflowBuilder`, `TypeGenerator`)
- **Functions/Methods**: `snake_case` (e.g., `extract_action_refs`, `build_all`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `CACHE_FILE`, `CONFIG_FILE`)
- **Action references**: `owner/repo@version` format (e.g., `actions/checkout@v5`)
- **Tests**: Inline with `#[cfg(test)]` blocks in each module
- **CLI output**: Uses `colored` crate for colored terminal output with emoji prefixes

## Configuration Files

| File | Purpose | Committed |
|------|---------|-----------|
| `Cargo.toml` | Rust dependencies and release profile | Yes |
| `gaji.config.ts` | Project config (dirs, watch settings, build options) | Yes |
| `gaji.config.local.ts` | Private config (GitHub token, custom API URL) | No |
| `mise.toml` | Dev environment (deno latest, node LTS) | Yes |
| `release-plz.toml` | Automated version bumping config | Yes |

## CI/CD

CI workflows live in `.github/workflows/` and are **generated from** `workflows/ci.ts` using gaji itself (self-dogfooding).

**PR checks** (`.github/workflows/pr.yml`):
- `cargo test --all-features`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dodok8/gaji](https://github.com/dodok8/gaji) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
