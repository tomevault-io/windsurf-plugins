---
trigger: always_on
description: **tomat** is a Pomodoro timer with daemon support designed for waybar and other
---

# LLM Agent Instructions for tomat

## Repository Overview

**tomat** is a Pomodoro timer with daemon support designed for waybar and other
status bars. It's a Rust project (~4,300 lines across multiple modules) that
implements a server/client architecture using Unix sockets for inter-process
communication.

**Key Details:**

- **Language:** Rust (2024 edition)
- **Architecture:** Client/server with Unix socket communication
- **Target:** Linux systems with systemd user services
- **Purpose:** Lightweight Pomodoro timer for waybar integration
- **Dependencies:** Standard Rust ecosystem (tokio, clap, serde, chrono,
  notify-rust, fs2, rodio)
- **Testing:** Comprehensive integration tests (37 tests across 7 modules
  covering all functionality)

## Build & Development Environment

### Prerequisites

- Rust stable toolchain (specified in `rust-toolchain.toml`)
- Cargo for building and dependency management
- Optional: Task runner (`go-task`) for development workflows
- Optional: Nix/devenv for reproducible development environment

### Essential Build Commands

**Always run commands from the repository root (`/home/jola/projects/tomat`).**

1. **Quick development check:**

   ```bash
   task dev
   ```

   This runs: `cargo check` → `cargo test` →
   `cargo clippy --all-targets --all-features -- -D warnings`

2. **Individual commands:**

   ```bash
   # Check compilation without building
   cargo check

   # Run tests (comprehensive integration test suite)
   cargo test

   # Run specific test categories by module
   cargo test --test cli integration::timer      # Timer behavior tests
   cargo test --test cli integration::daemon     # Daemon management tests
   cargo test --test cli integration::formats    # Output format tests
   cargo test --test cli integration::commands   # Command validation tests
   cargo test --test cli integration::hooks      # Hook execution tests

   # Lint with clippy - MUST pass with zero warnings
   cargo clippy --all-targets --all-features -- -D warnings

   # Check code formatting
   cargo fmt -- --check

   # Format code
   cargo fmt
   ```

3. **Build commands:**

   ```bash
   # Development build (fast)
   cargo build

   # Release build (optimized, ~1.2s from clean)
   cargo build --release

   # Clean build (from clean state takes ~10s for dependencies)
   cargo clean && cargo build
   ```

4. **Installation:**

   ```bash
   # Quick install with systemd service setup
   ./install.sh

   # Manual install
   cargo install --path .
   ```

### Pre-commit Validation

**CRITICAL:** All code changes MUST pass these checks before commit:

1. **Formatting:** `cargo fmt -- --check` (MUST exit with code 0)
2. **Linting:** `cargo clippy --all-targets --all-features -- -D warnings` (MUST
   exit with code 0, no warnings allowed)
3. **Compilation:** `cargo check` (MUST pass)
4. **Tests:** `cargo test` (37 integration tests must pass)

**Pre-commit hooks are configured** in `.pre-commit-config.yaml` and will run
clippy and rustfmt automatically if using the Nix devenv.

## Project Layout & Architecture

### File Structure

```
/
├── src/
│   ├── main.rs               # Entry point and command dispatching
│   ├── cli.rs                # CLI argument parsing with clap (command definitions)
│   ├── config.rs             # Configuration system (timer, sound, notification settings)
│   ├── server.rs             # Unix socket server, daemon logic, and process management
│   ├── timer.rs              # Timer state management, phase transitions, and notification system
│   └── audio.rs              # Sound playback system with embedded audio files
├── tests/
│   ├── cli.rs                # Integration test entry point
│   └── integration/          # Modular integration test modules
│       ├── mod.rs           # Module declarations
│       ├── common.rs        # Shared test utilities (TestDaemon helper)
│       ├── daemon.rs        # Daemon lifecycle tests
│       ├── timer.rs         # Timer behavior and auto-advance tests
│       ├── commands.rs      # Command validation tests
│       ├── formats.rs       # Output format tests
│       └── hooks.rs         # Hook execution tests
├── docs/
│   ├── book.toml             # mdbook configuration
│   └── src/                  # Documentation source (markdown)
│       ├── SUMMARY.md       # Navigation structure
│       ├── index.md         # Documentation index
│       ├── overview.md      # Architecture, quick start, examples
│       ├── configuration.md # Configuration guide
│       ├── integration.md   # Waybar, systemd, hooks
│       ├── troubleshooting.md # Common issues
│       └── cli-reference.md # Auto-generated from clap (DO NOT EDIT)
├── assets/
│   ├── icon.png              # Embedded notification icon
│   └── sounds/               # Embedded audio files
├── images/
│   ├── logo.svg              # Source logo (visual identity)
│   ├── logo.png              # Generated logo for GitHub/docs (256x256)
│   └── og.png                # Generated social media image (1280x640)
├── build.rs                  # Build script for man pages, mdbook, icons, completions
├── Cargo.toml               # Dependencies and metadata, includes cargo-deb config
├── Cargo.lock               # Dependency lockfile

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jolars/tomat](https://github.com/jolars/tomat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
