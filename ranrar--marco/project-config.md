---
trigger: always_on
description: Marco is a GTK4-based Rust markdown editor with nom-based parser. This guide helps AI agents understand the project's architecture and workflows.
---

# Marco Copilot Instructions

Marco is a GTK4-based Rust markdown editor with nom-based parser. This guide helps AI agents understand the project's architecture and workflows.

## Communication Style

When completing work, **DO NOT create markdown documentation files**. Instead:
- Write summaries directly in chat responses
- Use simple tables for data
- Keep text blocks small and focused
- Be concise and to-the-point

## Problem-Solving Approach

When facing an issue or problem:
1. **Review existing code** - Check how similar issues are handled elsewhere in the codebase
2. **Search online** - Use web search to find solutions, best practices, and documentation
3. **Analyze the problem** - Break down complex issues into smaller, manageable parts
4. **Test solutions** - Verify fixes work before considering the task complete

## Development Workflow

### Rust Toolchain
Marco uses **Rust 1.92.0** (stable) with the following components:
- **rustfmt** - Code formatting (`cargo fmt`)
- **clippy** - Linting and code quality (`cargo clippy`)
- **rust-src** - Source code for standard library (required for rust-analyzer)
- **rust-docs** - Standard library documentation (`rustup doc --std`)
- **llvm-tools** - LLVM utilities for profiling and code coverage

**Toolchain file**: `rust-toolchain.toml` pins the version across all machines

**Development commands**:
```bash
cargo fmt                    # Format code
cargo clippy                 # Run linter
cargo test --workspace       # Run all workspace tests
cargo doc --workspace --open # Generate & view project docs
cargo llvm-cov --html --open # Generate code coverage report
rustup doc                   # View Rust standard library docs
```

**Code coverage**: Use `cargo llvm-cov` to analyze test coverage. UI coverage is typically low/0% for GTK apps. Pure-Rust parser/render coverage now lives in the separate [marco-core](https://github.com/Ranrar/marco-core) repository.

### VS Code workspaces (native OS)

This repo intentionally supports **native per-OS development**. Use the workspace file that matches the OS you are on:

- **Linux**: `marco-linux.code-workspace`
- **Windows (MSVC)**: `marco-windows.code-workspace`

Avoid configuring Rust Analyzer to use `x86_64-pc-windows-gnu` on Linux for this project: GTK/Glib sys crates (e.g. `glib-sys`) rely on `pkg-config` and require a full Windows/MinGW sysroot for cross compilation, which will produce noisy diagnostics that are not actionable for most contributions.

### Using Logs for Testing
Marco uses file-based logging as part of the development workflow:
- **Run the application**: `cargo run -p marco` or `cargo run -p polo`
- **Check the log**: Open `log/YYYYMM/YYMMDD.log` (e.g., `log/202510/251007.log`)
- **Verify behavior**: Look for errors, warnings, or debug messages
- **Part of testing**: Reading logs is essential before marking work complete

## Architecture Overview

Marco uses a **Cargo workspace** with three crates and depends on the
externally-published [`marco-core`](https://crates.io/crates/marco-core) crate
(developed in its own repository: https://github.com/Ranrar/marco-core).

### Workspace Structure
- **`marco-shared/`** - Shared app logic: buffer management, settings, paths, loaders, layout state. No GTK dependencies. Also owns the centralized assets and the `build.rs` that copies them into `target/*/marco_assets/`.
- **`marco/`** - Full-featured editor binary: GTK4 UI, SourceView5 text editing, WebKit6 preview. Depends on `marco-core` (crates.io) and `marco-shared`.
- **`polo/`** - Lightweight viewer binary: GTK4 UI, WebKit6 preview only (no SourceView5). Depends on `marco-core` (crates.io) and `marco-shared`.
- **`marco-shared/src/assets/`** - Centralized assets: themes, fonts, icons, settings.

### Core Components

#### marco-core (external crate, separate repo)
The parser, AST, HTML renderer, and intelligence/LSP features live in the
`marco-core` crate published on crates.io. The pinned version is declared in
the workspace `Cargo.toml` under `[workspace.dependencies.marco-core]`.

Key modules (in the `marco-core` repo, accessible via the published crate):
- **`grammar/`** - nom-based grammar parsers for block and inline Markdown elements
- **`parser/`** - AST building from grammar output
- **`render/`** - HTML renderer with entity escaping and syntax highlighting support
- **`intelligence/`** (formerly `lsp/`) - syntax highlighting, diagnostics, completion, hover
- **`logic/`** - Pure Rust business logic: cache, logging

#### marco-shared Library (`marco-shared/src/`)
- **`logic/`** - Shared app logic: buffer management, settings, layout state, loaders
- **`paths/`** - Cross-platform path resolution for assets and config

#### marco Binary (`marco/src/`)
- **`components/editor/`** - GTK4 editor UI with SourceView5 integration  
- **`components/viewer/`** - WebKit6-based preview rendering
- **`components/language/`** - Localization support
- **`logic/`** - UI-specific logic: GTK signal management, menu handlers
- **`ui/`** - GTK widgets and split view layout
- **`ui/css/`** - Programmatic CSS generation system

#### polo Binary (`polo/src/`)
- Viewer-focused application (read-only viewer companion)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ranrar/Marco](https://github.com/Ranrar/Marco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
