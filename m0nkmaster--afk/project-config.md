---
trigger: always_on
description: **afk** - Autonomous AI coding loops, Ralph Wiggum style.
---

# Agent Instructions

**afk** - Autonomous AI coding loops, Ralph Wiggum style.

## Project Overview

This is a Rust CLI tool that implements the Ralph Wiggum pattern for autonomous AI coding. It aggregates tasks from multiple sources and generates prompts for AI coding tools.

## Development Setup

```bash
cargo build --release
cargo test -- --test-threads=1
```

## Issue Tracking

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Code Quality

Before committing, run:

```bash
cargo fmt -- --check        # Formatting
cargo clippy                # Linting
cargo test -- --test-threads=1  # Run tests
```

## Testing

This project maintains comprehensive tests. Tests are required for all changes.

Write tests first. Follow TDD where practical.

### Running Tests

**Important:** Tests must run single-threaded due to the `notify` crate's FSEvents backend on macOS. Parallel test execution can cause hangs.

```bash
cargo test -- --test-threads=1      # Run all tests (recommended)
RUST_TEST_THREADS=1 cargo test      # Alternative via env var
cargo test --release -- --test-threads=1  # Release mode
cargo test config:: -- --test-threads=1   # Run tests for config module
```

### Test Modules

Tests are inline with modules (`#[cfg(test)] mod tests`). Key test coverage:

| Module | Description |
|--------|-------------|
| `cli` | CLI parsing and command structure |
| `config` | Configuration loading and serialisation |
| `progress` | Session and task progress tracking |
| `bootstrap` | Project analysis and AI CLI detection |
| `prompt` | Tera template rendering |
| `prd` | PRD document parsing and management |
| `sources` | All source adapters (beads, json, markdown, github, openspec, gherkin) |
| `parser` | Output parsing with regex patterns |
| `feedback` | Metrics collection and ASCII art |
| `watcher` | File system monitoring |
| `runner` | Loop controller and iteration runner |
| `cli_integration` | End-to-end CLI command tests (in tests/) |

### Writing Tests

1. **Use tempfile** for temporary directories
2. **Mock external calls** via Command patterns where appropriate
3. **Test edge cases** - empty inputs, missing files, error conditions
4. **Keep tests focused** - one behaviour per test

## Architecture

```
src/
├── main.rs              # Entry point
├── lib.rs               # Library exports
├── path_matcher.rs      # Shared utility for ignore patterns
├── bootstrap/
│   └── mod.rs           # Project analysis, AI CLI detection
├── cli/
│   ├── mod.rs           # Clap CLI - commands and argument handling
│   ├── output.rs        # Output formatting utilities
│   ├── update.rs        # Self-update functionality
│   └── commands/
│       ├── mod.rs       # Command module exports
│       ├── archive.rs   # Archive session management
│       ├── completions.rs # Shell completions
│       ├── config.rs    # Config show/set commands
│       ├── go.rs        # Main loop command
│       ├── import.rs    # Import PRD/tasks
│       ├── init.rs      # Project initialisation
│       ├── progress_cmd.rs # Progress display
│       ├── prompt.rs    # Prompt preview
│       ├── source.rs    # Source management
│       ├── status.rs    # Status display
│       ├── task.rs      # Task management (done/fail/reset)
│       ├── use_cli.rs   # AI CLI switching
│       └── verify.rs    # Quality gate verification
├── config/
│   ├── mod.rs           # Serde models for .afk/config.json
│   ├── field.rs         # Config field definitions
│   ├── metadata.rs      # Config metadata handling
│   └── validation.rs    # Config validation rules
├── feedback/
│   ├── mod.rs           # Module exports
│   ├── art.rs           # ASCII art spinners and mascots
│   ├── celebration.rs   # Task/session completion displays
│   ├── display.rs       # Progress display panels
│   ├── metrics.rs       # Iteration metrics collection
│   └── spinner.rs       # Inline spinner animations
├── git/
│   └── mod.rs           # Git operations (commit, archive)
├── parser/
│   ├── mod.rs           # AI CLI output parsing (regex patterns)
│   └── stream_json.rs   # Streaming JSON parser for AI CLI output
├── prd/
│   ├── mod.rs           # PRD document model
│   ├── parse.rs         # PRD parsing
│   └── store.rs         # PRD persistence and sync
├── progress/
│   ├── mod.rs           # Session and task progress tracking
│   ├── archive.rs       # Archive logic for sessions
│   └── limits.rs        # Iteration limits and constraints
├── prompt/
│   ├── mod.rs           # Tera template rendering
│   └── template.rs      # Template utilities
├── runner/
│   ├── mod.rs           # Module exports
│   ├── controller.rs    # Loop lifecycle management
│   ├── iteration.rs     # Single iteration execution
│   ├── output_handler.rs # Console output
│   ├── quality_gates.rs # Lint, test, type checks
│   └── sleep_guard.rs   # System sleep prevention
├── sources/
│   ├── mod.rs           # aggregate_tasks() dispatcher
│   ├── beads.rs         # Beads (bd) integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m0nkmaster/afk](https://github.com/m0nkmaster/afk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
