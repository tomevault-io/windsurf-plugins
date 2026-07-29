---
trigger: always_on
description: This is a Rust project. Always consider Rust-specific idioms: always validate suggestions against Rust's ownership/borrowing rules before recommending removal of .clone(), .to_string(), or similar patterns, verify numeric types and bit widths before flagging values as errors, and suggest running `cargo clippy` when unsure about a suggestion.
---

# Claude Code Generation Guidelines for Rust Projects

## Language & Project Context

This is a Rust project. Always consider Rust-specific idioms: always validate suggestions against Rust's ownership/borrowing rules before recommending removal of .clone(), .to_string(), or similar patterns, verify numeric types and bit widths before flagging values as errors, and suggest running `cargo clippy` when unsure about a suggestion.

## Code Review

When asked to review a file, review the file directly by reading it. Do NOT attempt to use `gh` CLI, PR numbers, or any PR-based review workflow. Just read the file and provide a code review.

When providing code reviews, deliver the complete review in one response. Do not stop partway through. If the review is long, prioritize critical issues first, then minor suggestions.

## Project Overview

swyh-rs is a "Stream What You Hear" application: it captures audio from the system's default output device (via CPAL/WASAPI loopback on Windows, ALSA/PipeWire on Linux) and streams it over HTTP to DLNA/UPnP renderers discovered via SSDP. It has both a GUI binary (FLTK) and a headless CLI binary. Key subsystems:

- **Audio capture**: `src/audio/` — CPAL-based capture, sample conversion, FLAC/WAV/MP3 encoding
- **HTTP streaming server**: `src/server/` — tiny HTTP server that pushes the audio stream to renderers
- **Renderer control**: `src/rendercontrol/` — SSDP discovery and UPnP AV/OpenHome renderer control
- **UI**: `src/ui/mainform.rs` — FLTK tabbed GUI (Audio, Network, App, Status tabs)
- **Config**: `src/utils/configuration.rs` — INI-based persistent config via `serde`
- **CLI**: `src/bin/swyh-rs-cli.rs`, CLI args parsed with `lexopt`
- **i18n**: `src/utils/i18n.rs` — `fluent-templates` with `set_use_isolating(false)` (avoids bidi chars in FLTK labels)

## Core Architecture Principles

### 1. Error Handling & Resource Management

- **Use Result types**: Prefer `Result<T, E>` over panics for recoverable errors
- **Explicit error handling**: Use `?` operator and proper error propagation
- **RAII pattern**: Rust's ownership system handles resource cleanup automatically
- **Custom error types**: Create domain-specific error types using `thiserror` or `anyhow`

```rust
// Good example
use anyhow::{Context, Result};

fn process_file(path: &str) -> Result<String> {
    let content = std::fs::read_to_string(path)
        .with_context(|| format!("Failed to read file: {}", path))?;
    
    // Process content...
    Ok(content)
}
```

### 2. Concurrency & Thread Safety

- **Ownership model**: Leverage Rust's ownership system for thread safety
- **Thread-based concurrency**: This project uses OS threads, not async/await — do not introduce `tokio` or `async-std`
- **Channel communication**: Use `crossbeam-channel` for thread communication
- **Mutex/RwLock**: Use for shared mutable state when necessary

### 3. Configuration & Dependency Injection

- **Serde configuration**: Use `serde` for serialization/deserialization
- **Dependency injection**: Pass dependencies explicitly through constructors
- **Feature flags**: Use Cargo features for conditional compilation

## File and Directory Structure

### Standard Layout

```text
swyh-rs/
├── src/
│   ├── lib.rs
│   ├── bin/
│   │   ├── swyh-rs.rs         # GUI binary entry point
│   │   └── swyh-rs-cli.rs     # Headless CLI binary
│   ├── audio/                 # CPAL capture, sample conversion, encoding
│   ├── enums/                 # Shared message/streaming enums
│   ├── globals/               # Static globals and shared state
│   ├── rendercontrol/         # SSDP discovery, UPnP/OpenHome control
│   ├── server/                # HTTP streaming server
│   ├── ui/                    # FLTK GUI (mainform.rs)
│   └── utils/                 # Config, CLI parsing, i18n, logging, etc.
├── assets/                    # Icons and static assets
├── locales/                   # Fluent (.ftl) i18n files
├── tray_icon/                 # Windows tray icon helper (Python)
├── Cargo.toml
└── Cargo.lock
```

### File Naming Conventions

- **Rust files**: Use snake_case (e.g., `user_service.rs`, `auth_handler.rs`)
- **Test files**: Integration tests in `tests/` directory
- **Module files**: `mod.rs` for module declarations
- **Binary targets**: Place in `src/bin/` for additional executables

## Code Style & Standards

### Documentation

- **Rustdoc comments**: Use `///` for public API documentation
- **Module documentation**: Document modules with `//!` at the top
- **Examples**: Include code examples in documentation
- **Cargo.toml metadata**: Include proper project metadata

```rust
/// Processes user authentication requests.
///
/// # Arguments
///
/// * `username` - The user's username
/// * `password` - The user's password
///
/// # Returns
///
/// Returns `Ok(User)` if authentication succeeds, or `Err(AuthError)` if it fails.
///
/// # Examples
///
/// ```
/// let user = authenticate("alice", "secret123")?;
/// println!("Welcome, {}!", user.name);
/// ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dheijl/swyh-rs](https://github.com/dheijl/swyh-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
