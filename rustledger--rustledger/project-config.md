---
trigger: always_on
description: This document provides context for AI assistants working on the rustledger codebase.
---

# Rustledger Development Guidelines

This document provides context for AI assistants working on the rustledger codebase.

## Project Overview

Rustledger is a pure Rust implementation of Beancount, the double-entry bookkeeping language. It provides a 10-30x faster alternative to Python beancount with full syntax compatibility.

## Architecture

The project is a Cargo workspace with these crates:

| Crate | Purpose |
|-------|---------|
| `rustledger-core` | Core types (Amount, Position, Inventory, Directives) |
| `rustledger-parser` | Lexer and parser with error recovery |
| `rustledger-loader` | File loading, includes, options |
| `rustledger-booking` | Interpolation and booking engine (7 methods) |
| `rustledger-validate` | Validation with 27 error codes |
| `rustledger-query` | BQL query engine |
| `rustledger-plugin` | Native and WASM plugin system (20 plugins) |
| `rustledger-importer` | Import framework for bank statements |
| `rustledger` | CLI tool (`rledger check`, `rledger query`, etc.) |
| `rustledger-wasm` | WebAssembly library target |
| `rustledger-lsp` | Language Server Protocol implementation |
| `rustledger-ffi-wasi` | FFI via WASI for embedding in any language |

## Rust-Specific Patterns

### Error Handling Strategy

This project uses a layered error handling approach:

| Layer | Crate | Pattern |
|-------|-------|---------|
| Library crates | `rustledger-*` | `thiserror` with typed errors |
| CLI binaries | `rustledger` | `anyhow` for ergonomic error chains |
| Tests | All | `anyhow` or `.unwrap()` with clear context |

**Error type conventions:**

```rust
// In library code - use thiserror
#[derive(Debug, thiserror::Error)]
pub enum ParseError {
    #[error("unexpected token at {location}: expected {expected}, found {found}")]
    UnexpectedToken { location: Span, expected: String, found: String },

    #[error("invalid amount: {0}")]
    InvalidAmount(#[from] DecimalError),
}

// In CLI code - use anyhow
fn main() -> anyhow::Result<()> {
    let ledger = load_file(&path).context("failed to load ledger")?;
    Ok(())
}
```

**Rules:**

- Never use `.unwrap()` in library code unless mathematically impossible to fail
- Use `.expect("reason")` only when failure indicates a bug
- Propagate errors with `?`, add context with `.context()` or `.with_context()`
- User-facing errors must include file location (path, line, column)

### Async Runtime

This project is **synchronous** - no async runtime is used. Reasons:

- File I/O is the only blocking operation, and it's fast enough sync
- Simpler mental model for accounting calculations
- WASM target doesn't support async well

**If async is ever needed:**

- Use `tokio` (not async-std)
- Prefer `tokio::fs` over `std::fs` in async contexts
- Use `#[tokio::main]` for CLI, `#[tokio::test]` for async tests

### Memory Management Guidelines

| Type | When to Use |
|------|-------------|
| `&str` | Borrowed string, no ownership needed |
| `String` | Owned string, needs modification or storage |
| `Cow<'a, str>` | May or may not need to own (parsing, transformations) |
| `Arc<str>` | Shared ownership across threads (interned strings) |
| `Box<T>` | Single ownership of heap data, rare in this codebase |
| `Rc<T>` | **Avoid** - not thread-safe, use `Arc` instead |

**String interning:**
The project uses string interning for frequently repeated values:

- Account names (`Assets:Bank:Checking`)
- Currency codes (`USD`, `EUR`)
- Payee names

Interned strings are stored as `Arc<str>` in a global interner. Use `intern_string()` to intern.

**Allocation guidelines:**

- Prefer `SmallVec<[T; N]>` for small, bounded collections (e.g., posting legs)
- Use `Vec::with_capacity()` when size is known
- Avoid `clone()` in hot paths - prefer borrowing
- Use `Cow` for functions that sometimes need to modify strings

### Type-First Development

When implementing new features:

1. **Define types first** - structs, enums, traits
1. **Write trait bounds** - what capabilities are needed?
1. **Let the compiler guide you** - fix errors iteratively
1. **Implement logic last** - once types compile, logic is constrained

This approach works well with AI assistance because:

- Rust's compiler provides structured feedback
- Type errors are specific and actionable
- The AI can iterate quickly on compiler output

## Code Standards

### Rust Idioms

- Use `Result<T, E>` for fallible operations, not panics
- Prefer `?` operator over `.unwrap()` in production code
- Use `thiserror` for error types, `anyhow` in CLI/tests only
- Prefer iterators over explicit loops where idiomatic
- Use `#[must_use]` on functions returning important values
- Prefer `&str` over `String` when possible
- Use `Cow<'a, str>` for potentially-owned strings

### Testing

| Test Type | Location | Framework |
|-----------|----------|-----------|
| Unit tests | `#[cfg(test)]` in source files | `#[test]` |
| Integration tests | `crates/*/tests/*.rs` | `#[test]` |
| Snapshot tests | Parser output, error messages | `insta` |
| Property tests | Invariants, roundtrips | `proptest` |
| Fuzzing | Parser, untrusted input | `cargo-fuzz` (nightly) |

**Test naming:** `test_<function>_<scenario>`

```rust
#[test]
fn test_parse_amount_with_commodity() { }

#[test]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustledger/rustledger](https://github.com/rustledger/rustledger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
