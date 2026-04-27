---
trigger: always_on
description: This file provides guidance to Google Gemini and other AI coding assistants when working with code in this repository.
---

# GEMINI.md

This file provides guidance to Google Gemini and other AI coding assistants when working with code in this repository.

## Project Overview

Claude Token Counter is a Rust-based CLI tool that helps users monitor their Claude token usage through two approaches: (1) Real-time parsing of local Claude Code JSONL logs for all users, and (2) API-based tracking for Team/Enterprise users with Admin keys. The tool provides accurate cost estimation, beautiful terminal visualization, and comprehensive token tracking across all token types (input, output, cache creation, cache read).

## Current Workflow Phase

**Phase**: Core Implementation Complete
**Status**: Beta - Live monitoring fully functional, API integration complete, ready for enhancements

### Workflow Checklist

- [x] Project initialization with Cargo
- [x] Dependency configuration
- [x] Basic CLI structure with subcommands
- [x] Documentation suite (README, development guide)
- [x] Git repository setup and GitHub integration
- [x] SSH authentication configuration
- [x] Configuration module implementation
- [x] API client module implementation
- [x] Data models definition
- [x] Status command implementation
- [x] History command implementation
- [x] Config command implementation
- [x] Local JSONL parsing module
- [x] Live monitoring command
- [x] Cost calculation engine
- [x] Terminal formatting and display
- [x] Error handling and validation
- [ ] File watching (instant updates)
- [ ] Filtering by date/project/model
- [ ] Export functionality (CSV/JSON)
- [ ] Unit and integration tests
- [ ] Cross-platform testing (Linux/Windows)
- [ ] CI/CD automation
- [ ] Pre-built binary releases

## Build and Development Commands

### Building
```bash
# Development build
cargo build

# Release build (optimized)
cargo build --release

# The binary will be at target/release/claude-token-counter
```

### Running
```bash
# Run directly with cargo
cargo run -- live                    # Live monitoring (recommended)
cargo run -- live --refresh 5        # Custom refresh interval
cargo run -- status                  # API-based current status
cargo run -- history --days 7        # API-based history
cargo run -- config --api-key KEY    # Configure API key

# Run the built binary
./target/release/claude-token-counter live
```

### Testing
```bash
# Run all tests
cargo test

# Run tests with output
cargo test -- --nocapture

# Run specific test
cargo test test_name
```

### Code Quality
```bash
# Format code
cargo fmt

# Check formatting without modifying
cargo fmt -- --check

# Run clippy (linter)
cargo clippy

# Run clippy with all warnings
cargo clippy -- -W clippy::all
```

### Development Workflow
```bash
# Check code compiles without building
cargo check

# Build and run in one step
cargo run -- live

# Watch mode (requires cargo-watch)
cargo watch -x "run -- live"
```

## Architecture

### High-Level Design

The application follows a modular architecture with dual data sources:

1. **CLI Layer** (`main.rs`): Command-line interface using `clap` with derive macros. Defines four subcommands (status, history, config, live) and handles argument parsing. Contains live monitor implementation with terminal control.

2. **Local Module** (`src/local/`): Parses Claude Code JSONL logs from `~/.claude/projects/`. Recursively discovers all JSONL files, deserializes log entries, extracts token usage data, and aggregates statistics. Works for all users without requiring API keys.

3. **API Client Module** (`src/api/`): Handles communication with Anthropic Usage API. Supports two endpoints for regular API usage and Claude Code specific usage. Requires Admin API keys (Team/Enterprise only).

4. **Configuration Module** (`src/config/`): Manages API key storage and user preferences. Stores config in `~/.config/claude-token-counter/config.json` using the `dirs` crate for cross-platform home directory resolution.

5. **Display Module** (`src/display/`): Formats and presents data to the terminal using `colored` for styled output. Includes status display, history visualization, progress bars, and formatted tables.

6. **Models Module** (`src/models/`): Serde-based structs for API responses including UsageRecord, UsageResponse, and UsageSummary for aggregated statistics.

### Key Design Decisions

- **Dual-Mode Architecture**: Supports both local file parsing (universal) and API integration (enterprise), making the tool valuable for all user types
- **Local-First Approach**: Prioritizes local JSONL parsing as the primary feature since it works for all users and provides more granular data
- **Async Runtime**: Uses Tokio for async operations to handle API requests and non-blocking sleep in live monitor
- **Real-Time Monitoring**: Implements live refresh using crossterm for terminal control and tokio::sleep for intervals
- **Comprehensive Token Tracking**: Tracks all token types (input, output, cache creation, cache read) for accurate cost calculation
- **Error Handling**: Uses `anyhow` for flexible error handling with context. Gracefully handles malformed JSONL lines
- **Config Storage**: Stores API keys securely in user's config directory (never in the repository)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rubenmendoza1290/claude-token-counter](https://github.com/rubenmendoza1290/claude-token-counter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
