---
trigger: always_on
description: This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a high-performance Rust CLI tool designed as a Claude Code hook for displaying cross-platform desktop notifications with advanced sound support. The tool integrates seamlessly with Claude Code's hooks system to provide immediate user feedback during development sessions.

## Build Commands

### Using Make (Recommended)

The project includes a comprehensive Makefile with optimized commands:

- `make build-release` - Compile optimized release binary with full optimizations
- `make install` - Build and install CLI globally as `claude-code-notification`
- `make test` - Run complete test suite with coverage
- `make fmt` - Format Rust code using rustfmt with project settings
- `make clippy` - Lint Rust code with clippy for code quality
- `make clean` - Clean all build artifacts and target directory
- `make help` - Show all available make targets with descriptions

### Using Cargo Directly

For direct cargo usage when Make is unavailable:

- `cargo build` - Compile debug binary with symbols
- `cargo build --release` - Compile optimized release binary
- `cargo run` - Run CLI from source with development settings
- `cargo install --path .` - Install CLI globally from current directory
- `cargo test` - Run test suite with default settings
- `cargo fmt` - Format Rust code with default configuration
- `cargo clippy` - Run clippy linter with default rules

## Usage as a Claude Code Hook

### Hook Configuration

This program integrates with Claude Code's notification system through the hooks configuration. Configure in your Claude Code settings file:

**Basic Configuration:**
```json
{
  "hooks": {
    "Notification": [
      {
        "type": "command",
        "command": "claude-code-notification"
      }
    ]
  }
}
```

**Advanced Configuration with Custom Sound:**
```json
{
  "hooks": {
    "Notification": [
      {
        "type": "command", 
        "command": "claude-code-notification --sound Submarine"
      }
    ]
  }
}
```

### JSON Input Schema

The hook receives structured JSON input via stdin:

```json
{
  "session_id": "string - Claude session identifier", 
  "transcript_path": "string - Path to session transcript file",
  "message": "string - Notification body text",
  "title": "string? - Optional notification title (defaults to 'Claude Code')"
}
```

## CLI Parameters and Sound System

### Sound Parameter Options

The `--sound` parameter supports intelligent path resolution:

**System Sounds** (recommended for consistency):
- Format: `--sound {SoundName}` (no path separators)
- Resolves to: `/System/Library/Sounds/{SoundName}.aiff`
- Available: Glass (default), Submarine, Frog, Purr, Basso, Blow, Bottle, Funk, Hero, Morse, Ping, Pop, Sosumi, Tink

**Custom Audio Files** (for specialized notifications):
- Format: `--sound {/path/to/file}` (contains path separators)
- Supports: `.wav`, `.aiff`, `.mp3`, `.m4a`, and other `afplay`-compatible formats
- Examples:
  - `--sound ./assets/notification.wav`
  - `--sound /Users/dev/sounds/alert.m4a`
  - `--sound ~/Music/custom-alert.aiff`

## Development Workflow

### Local Development

**Quick Development Cycle:**
```bash
# Run with immediate feedback
cargo run

# Run with custom sound for testing
echo '{"session_id":"dev","transcript_path":"/tmp/dev.md","message":"Development test","title":"Dev Test"}' | cargo run -- --sound Submarine
```

**Testing and Quality Assurance:**
```bash
# Run comprehensive tests
make test

# Check code formatting
make fmt

# Run linter for code quality
make clippy

# Full quality check pipeline
make test && make fmt && make clippy
```

### Manual Testing Scenarios

**Basic Functionality Testing:**
```bash
# Test default configuration
echo '{"session_id":"test","transcript_path":"/tmp/test.md","message":"Default notification test","title":"Basic Test"}' | cargo run

# Test system sound variants
echo '{"session_id":"test","transcript_path":"/tmp/test.md","message":"System sound test","title":"Sound Test"}' | cargo run -- --sound Glass
echo '{"session_id":"test","transcript_path":"/tmp/test.md","message":"Submarine sound test","title":"Sound Test"}' | cargo run -- --sound Submarine

# Test custom audio files
echo '{"session_id":"test","transcript_path":"/tmp/test.md","message":"Custom sound test","title":"Custom Test"}' | cargo run -- --sound ./366102__original_sound__confirmation-upward.wav
```

**Error Handling Testing:**
```bash
# Test invalid JSON handling
echo '{"invalid": json}' | cargo run 2>&1 | head -5

# Test missing sound file handling  
echo '{"session_id":"test","transcript_path":"/tmp/test.md","message":"Missing sound test","title":"Error Test"}' | cargo run -- --sound /nonexistent/file.wav
```

## Architecture and Implementation

### Project Structure

The codebase follows Rust best practices with clear separation of concerns:

- **`src/main.rs`** - CLI entry point with `clap` argument parsing and error handling
- **`src/lib.rs`** - Core notification logic, sound management, and parallel execution
- **`src/error.rs`** - Structured error types with `thiserror` for comprehensive error handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wyattjoh/claude-code-notification](https://github.com/wyattjoh/claude-code-notification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
