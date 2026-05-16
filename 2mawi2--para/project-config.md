---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Para is a "Parallel IDE Workflow Helper" - a Rust implementation that enables developers to work on multiple features simultaneously using Git worktrees and their favorite IDEs. It's specifically designed for AI-assisted development workflows, allowing multiple Claude Code instances to work on different branches without conflicts.

## Essential Development Commands

### Setup and Testing
```bash
just test             # Run all checks (tests + linting + formatting) - ALWAYS use this
just test <filter>    # Run specific tests only (skips linting/formatting for speed)
                     # Examples: just test finish, just test integration, just test core::git
just lint             # Run clippy linting for all files
just fmt              # Auto-fix Rust formatting
just build            # Build debug binary (with bun/npm fallback for MCP server)
just build-release    # Build optimized release binary
```

### Common Development Tasks
```bash
just install          # Install Rust para binary globally
just uninstall        # Remove para from system
just setup-hooks      # Configure git pre-commit/pre-push hooks
just status           # Check dependencies and project health
just clean            # Clean development artifacts
just release          # Create new release (patch version bump)
```

**Testing Guidelines:**
- **Always use `just test`** for final verification (includes linting + formatting)
- Use `just test <filter>` during development for faster iteration on specific tests
- Use `just lint` to check linting on all modules
- The `just` commands have clean, focused output - avoid raw `cargo` commands

### TypeScript/MCP Testing
```bash
just test-ts          # Run TypeScript tests for MCP server
just lint-ts          # Run ESLint on TypeScript code
just fmt-ts           # Format TypeScript code
```

## Architecture Overview

### Modular Rust Design
```
src/
├── cli/              # Command-line interface, argument parsing, and command implementations
│   └── commands/     # Individual command implementations (start, finish, resume, etc.)
│       ├── mcp/      # MCP server management (init, strategies for finding servers)
│       ├── list/     # Session listing with formatters and analyzers
│       └── resume/   # Session resumption with context and task transformation
├── config/           # Configuration management, validation, and interactive wizard
├── core/             # Core business logic, session management, and IDE integration
│   ├── git/          # Git operations, worktree management, and repository handling
│   │   └── diff.rs   # Git diff calculation and statistics
│   └── status.rs     # Session status tracking and reporting
├── ui/               # Terminal UI components
│   └── monitor/      # Interactive session monitoring UI
│       ├── coordinator.rs    # Main UI coordination
│       ├── renderer.rs       # UI rendering logic
│       ├── state_manager.rs  # State management
│       └── actions.rs        # UI action handling
├── utils/            # Utility functions, error handling, and helper modules
│   └── validation.rs # Session name validation
└── main.rs           # Application entry point
```

### TypeScript MCP Server
```
mcp-server-ts/        # Model Context Protocol server implementation
├── src/
│   ├── para-mcp-server.ts      # Main MCP server implementation
│   └── para-mcp-server.test.ts # MCP server tests
├── package.json      # Node.js dependencies
├── tsconfig.json     # TypeScript configuration
└── eslint.config.js  # ESLint configuration
```

### Key Dependencies
- **clap**: Command-line argument parsing with completion support
- **serde/serde_json**: Configuration serialization
- **dialoguer**: Interactive prompts and wizards
- **chrono**: Date/time handling for sessions
- **anyhow/thiserror**: Error handling
- **directories**: Cross-platform config directories
- **ratatui**: Terminal UI framework for monitor command
- **crossterm**: Cross-platform terminal manipulation

### Key Features
- **Context-Aware**: Auto-detects current session from working directory
- **Auto-Staging**: Automatically stages all changes during finish operations
- **Recovery System**: Session snapshots for later recovery with `para recover`
- **File Input Support**: `para start --file prompt.txt` for complex prompts from files
- **IDE Wrapper Mode**: Claude Code can run inside VS Code/Cursor terminals
- **Interactive Monitor**: Real-time session monitoring with `para monitor` command
- **Status Tracking**: Update and view session status for AI agents with `para status`
- **MCP Integration**: Built-in TypeScript MCP server for Claude Code integration
- **Custom Branch Names**: Support for `--branch` flag in finish command

## Para Workflow Preferences

**My preferences for this project:**
- Use `para finish` workflow - create branches for manual review before merging
- Dispatched agents should use `para finish "<commit message>"` to create branches for review

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2mawi2/para](https://github.com/2mawi2/para) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
