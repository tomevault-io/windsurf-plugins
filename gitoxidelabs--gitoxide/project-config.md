---
trigger: always_on
description: This repository contains `gitoxide` - a pure Rust implementation of Git. This document provides guidance for GitHub Copilot when working with this codebase.
---

# Copilot Instructions for Gitoxide

This repository contains `gitoxide` - a pure Rust implementation of Git. This document provides guidance for GitHub Copilot when working with this codebase.

## Project Overview

- **Language**: Rust (MSRV documented in gix/Cargo.toml)
- **Structure**: Cargo workspace with multiple crates (gix-\*, gitoxide-core, etc.)
- **Main crates**: `gix` (library entrypoint), `gitoxide` binary (CLI tools: `gix` and `ein`)
- **Purpose**: Provide a high-performance, safe Git implementation with both library and CLI interfaces

## Development Practices

### Test-First Development

- Protect against regression and make implementing features easy
- Keep it practical - the Rust compiler handles mundane things
- Use git itself as reference implementation; run same tests against git where feasible
- Never use `.unwrap()` in production code, avoid it in tests in favor of `.expect()` or `?`. Use `gix_testtools::Result` most of the time.
- Use `.expect("why")` with context explaining why expectations should hold, but only if it's relevant to the test.

### Error Handling

- Handle all errors, never `unwrap()`
- Provide error chains making it easy to understand what went wrong
- Binaries may use `anyhow::Error` exhaustively (user-facing errors)

#### `gix-error` (preferred for plumbing crates)

Plumbing crates are migrating from `thiserror` enums to `gix-error`. Check whether a crate already
uses `gix-error` (look at its `Cargo.toml`); if it does, follow the patterns below. If it still uses
`thiserror`, keep using `thiserror` for consistency within that crate.

- **Error type alias**: `pub type Error = gix_error::Exn<gix_error::Message>;`
- **Static messages**: `gix_error::message("something failed")`
- **Formatted messages**: `gix_error::message!("failed to read {path}")`
- **Wrapping callee errors with context**: `.or_raise(|| message("context about what failed"))?`
- **Standalone error (no callee)**: `Err(message("something went wrong").raise())`
- **Wrapping an `impl Error` with context**: `err.and_raise(message("context"))`
- **Closure/callback bounds**: use `Result<T, Exn>` (bare), not `Exn<Message>`;
  inside the function, convert with `.or_raise(|| message("..."))?`;
  inside the closure, convert typed to bare with `.or_erased()`
- **`Exn<E>` does NOT implement `std::error::Error`** — this is by design.
  - To convert: use `.into_error()` to get `gix_error::Error` (which does implement `std::error::Error`)
  - Example: `std::io::Error::other(exn.into_error())`
- **In tests** returning `gix_testtools::Result` (= `Result<(), Box<dyn Error>>`), `Exn` can't be used
  with `?` directly — use `.map_err(|e| e.into_error())?`
- **Common imports**: `use gix_error::{message, ErrorExt, ResultExt};`
- See `gix-error/src/lib.rs` module docs for a full migration guide from `thiserror`

### Commit Messages

Follow "purposeful conventional commits" style:

- Use conventional commit prefixes ONLY if message should appear in changelog
- Breaking changes MUST use suffix `!`: `change!:`, `remove!:`, `rename!:`
- Features/fixes visible to users: `feat:`, `fix:`
- Refactors/chores: no prefix (don't affect users)
- Examples:
  - `feat: add Repository::foo() to do great things. (#234)`
  - `fix: don't panic when calling foo() in a bare repository. (#456)`
  - `change!: rename Foo to Bar. (#123)`

### Code Style

- Follow existing patterns in the codebase
- No `.unwrap()` - use `.expect("context")` if you are sure this can't fail.
- Prefer references in plumbing crates to avoid expensive clones
- Avoid calling `.detach()` unless an owned value is explicitly required. Many `gix` APIs accept attached ids and references directly, so prefer keeping repository-backed handles like `gix::Id` when possible.
- Use `gix_features::threading::*` for interior mutability primitives

### Path Handling

- Paths are byte-oriented in git (even on Windows via MSYS2 abstraction)
- Use `gix::path::*` utilities to convert git paths (`BString`) to `OsStr`/`Path` or use custom types

## Building and Testing

### Quick Commands

- `just test` - Run all tests, clippy, journey tests, and try building docs
- `just check` - Build all code in suitable configurations
- `just clippy` - Run clippy on all crates
- `cargo test` - Run unit tests only

### Build Variants

- `cargo build --release` - Default build (big but pretty, ~2.5min)
- `cargo build --release --no-default-features --features lean` - Lean build (~1.5min)
- `cargo build --release --no-default-features --features small` - Minimal deps (~46s)

### Test Best Practices

- Run tests before making changes to understand existing issues
- Use `GIX_TEST_IGNORE_ARCHIVES=1` when testing on macOS/Windows
- Journey tests validate CLI behavior end-to-end
- Fixture scripts should document what behavior they exercise and what makes
  the fixture special. Leave clear-text breadcrumbs so future readers can tell
  which details are essential to the test. Use markdown doc-strings when available.
- Stabilize fixtures whose generated contents can vary by using the
  `_needs_archive` variants of functions in `gix-testtools`; these always use

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GitoxideLabs/gitoxide](https://github.com/GitoxideLabs/gitoxide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
