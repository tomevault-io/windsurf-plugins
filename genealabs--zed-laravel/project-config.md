---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Zed editor extension that provides Laravel development support, similar to the Laravel VSCode extension. The extension is written in Rust and aims to provide features such as:

- Clickable "go-to-definition" for Blade templates
- Clickable "go-to-definition" for Livewire components
- Clickable "go-to-definition" for Flux components
- Other Laravel-specific IDE features

**Important**: This is a learning project. The developer is learning Rust while building this extension, so explanations of Rust concepts, providing options, and teaching best practices are essential.

## Development Commands

Zed extensions are typically developed using:

```bash
# Build the extension (assuming standard Rust project)
cargo build

# Run tests
cargo test

# Check code without building
cargo check

# Format code
cargo fmt

# Run linter
cargo clippy

# Build for release
cargo build --release
```

**IMPORTANT - Binary for Local Development:**
The `.zed/settings.json` configures Zed to use the local build directly:
```json
{
  "lsp": {
    "laravel-lsp": {
      "binary": {
        "path": "laravel-lsp/target/release/laravel-lsp"
      }
    }
  }
}
```

Development workflow:
```bash
cargo build --release
# Then in Zed: Cmd+Shift+P → "zed: reload extensions"
```

No copying or symlinks needed - Zed reads the binary path from settings.

## Running Diagnostics (Important for Zed)

When using Claude Code in Zed, it doesn't have direct access to LSP diagnostics. Always run these commands to check for errors:

### Check for Compilation Errors
```bash
cargo check
```
This is the fastest way to check if your code compiles without actually building the binary. Run this frequently while developing.

### See Detailed Compiler Messages
```bash
cargo build
```
This compiles the project and shows all errors and warnings with detailed explanations. The Rust compiler gives very helpful error messages - always read them carefully!

### Run Clippy for Best Practice Lints
```bash
cargo clippy
```
Clippy is Rust's linter that catches common mistakes and suggests more idiomatic code. Very useful when learning Rust!

### Run Tests
```bash
cargo test
```
Runs all tests in the project. Add `-- --nocapture` to see println! output during tests.

### Format Code
```bash
cargo fmt
```
Automatically formats your code according to Rust style guidelines. Run this before committing.

### Install the Extension Locally in Zed
```bash
# Install for local development/testing
zed: install dev extension
```
Use this command within Zed to load your extension for testing.

**Important**: After making changes, always run `cargo check` or `cargo build` to see if your code compiles before proceeding with more changes.

## Zed Extension Architecture

Zed extensions follow the Extension API provided by Zed. Key concepts:

- Extensions are written in Rust (or can use WebAssembly)
- Extensions interact with the Zed editor through the Extension API
- Language features like "go-to-definition" are typically implemented using the Language Server Protocol (LSP)
- Extensions can provide custom language servers or enhance existing ones

## Laravel-Specific Features to Implement

### Go-to-Definition Targets

1. **Blade Components**: `<x-component-name>` → `resources/views/components/component-name.blade.php`
2. **Livewire Components**: `<livewire:component-name>` → `app/Livewire/ComponentName.php`
3. **Flux Components**: `<flux:component>` → Flux component definition
4. **View References**: `view('view.name')` → `resources/views/view/name.blade.php`
5. **Route Names**: `route('route.name')` → route definition in `routes/` files
6. **Config References**: `config('app.name')` → `config/app.php`

## Architecture Notes

- Zed extensions MUST be written in Rust (compiled to WebAssembly)
- JavaScript/TypeScript cannot be used - VSCode extensions cannot be wrapped or ported
- Zed uses tree-sitter for syntax parsing
- May need custom tree-sitter queries for Laravel-specific patterns
- Extensions use the `zed_extension_api` crate and implement the `Extension` trait
- Language features use LSP (Language Server Protocol) integration

## LSP Architecture (laravel-lsp/)

### Core Components

| File | Purpose |
|------|---------|
| `main.rs` | LSP server, request handlers, Backend trait impl |
| `salsa_impl.rs` | Salsa incremental computation actor |
| `queries.rs` | Tree-sitter queries for pattern extraction |
| `parser.rs` | PHP and Blade tree-sitter parsing |
| `config.rs` | Laravel project configuration discovery |
| `env_parser.rs` | .env file parsing |
| `service_provider_analyzer.rs` | Middleware/binding extraction |
| `middleware_parser.rs` | Kernel.php and bootstrap/app.php parsing |

### Salsa Actor Pattern

The LSP uses a dedicated thread for Salsa incremental computation to avoid lifetime issues with async code:

```
┌─────────────────┐     oneshot channel     ┌─────────────────┐
│  LSP Handlers   │ ──────────────────────► │   SalsaActor    │
│  (async/await)  │ ◄────────────────────── │ (dedicated      │
│                 │        response         │  thread)        │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GeneaLabs/zed-laravel](https://github.com/GeneaLabs/zed-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
