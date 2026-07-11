---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow - Feature Branches

**IMPORTANT**: Now that the project is stable, we use feature branches for all development:

1. **Before starting any work**, create a feature branch:
   ```bash
   git checkout -b feature/descriptive-name
   # Examples: feature/add-default-impls, fix/batch-api-errors, docs/improve-examples
   ```

2. **Commit regularly** as you work:
   - After each logical change or set of related edits
   - Use clear, descriptive commit messages
   - Example: `git commit -m "Add Default impl for UpdateMemoryBlockRequest"`

3. **When feature is complete**, create a pull request to main
   - This keeps main stable and CI runs only on complete changes
   - Allows for code review and discussion

4. **Branch naming conventions**:
   - `feature/` - New features or enhancements
   - `fix/` - Bug fixes
   - `docs/` - Documentation improvements
   - `refactor/` - Code refactoring
   - `test/` - Test additions or improvements

## Development Principles

- Run `cargo check` frequently when producing code. This will help you catch errors early.
- NEVER use `unsafe{}`. If you feel you need to, stop, think about other ways, and ask the user for help if needed.
- NEVER ignore a failing test or change a test to make your code pass
- NEVER ignore a test
- ALWAYS fix compile errors before moving on.
- **ALWAYS ENSURE that tests will fail (via assert or panic with descriptive message) on any error condition**
- Use proper error handling with detailed context (LettaError types)
- Follow existing patterns for consistency
- Verify API behavior with curl when implementing new endpoints
- Use the web or context7 to help find docs, in addition to any other reference material
- Check TypeScript/Python SDKs for API patterns and expected behavior
- Test with local Letta server before assuming implementation is correct


## Project Overview

This is the development guide for contributing to letta, a Rust client library for the Letta REST API. This document contains internal implementation details, coding standards, and development workflows. For user documentation, see README.md.

### Implementation Status

**All 20 planned APIs are implemented.** Voice API is in beta with generic JSON support due to undocumented structure.

#### Pagination Implementation Matrix
- **Full pagination**: agents, messages, memory.archival, tags, providers, tools, sources.files, sources.passages, identities
- **No pagination**: blocks, groups, jobs, runs, batch (API limitations)
- **Generic types used**: Voice (serde_json::Value), Identities properties (serde_json::Value)

## Development Commands

```bash
# Quick start
nix develop           # Enter dev shell with all tools
just watch           # Auto-recompile on changes
just test           # Run all tests
just pre-commit-all # Format and lint

# Local Letta server for testing
cd local-server && docker compose up -d

# CLI tool (fully functional with all API operations)
cargo run --features cli -- --help
```

## Architecture

### Project Structure
- `src/lib.rs` - Library entry point
- `src/client.rs` - Main Letta client implementation
- `src/types/` - Data models for Letta API
  - `agent.rs` - Agent-related types and enums
  - `memory.rs` - Memory block types
  - `message.rs` - Message types
  - `tool.rs` - Tool types
  - `source.rs` - Source types
  - `groups.rs` - Group conversation types
  - `project.rs` - Project management types
  - `template.rs` - Agent template types
  - `runs.rs` - Execution run types
  - `health.rs` - Health check types
  - `models.rs` - Model configuration types
  - `tags.rs` - Tag-related types
  - `provider.rs` - Provider configuration types
  - `identities.rs` - Identity types (User, Org, Other)
  - `batch.rs` - Batch processing types
  - `telemetry.rs` - Telemetry trace types
  - `common.rs` - Shared types and utilities
- `src/api/` - API endpoint implementations
  - `agents.rs` - Agent CRUD operations
  - `messages.rs` - Message handling with agents
  - `memory.rs` - Core and archival memory operations
  - `sources.rs` - Document and data source management
  - `tools.rs` - Tool management
  - `blocks.rs` - Memory block operations
  - `groups.rs` - Multi-agent group conversations
  - `jobs.rs` - Asynchronous job management
  - `projects.rs` - Project management
  - `templates.rs` - Agent templates
  - `runs.rs` - Execution runs
  - `health.rs` - Health checks
  - `models.rs` - Model configuration endpoints
  - `tags.rs` - Tag management
  - `providers.rs` - Provider management
  - `misc.rs` - Miscellaneous endpoints
  - `batch.rs` - Batch processing operations
  - `telemetry.rs` - Telemetry trace retrieval
  - `voice.rs` - Voice conversation support (beta)
- `src/cli.rs` - CLI testing tool (binary: `letta`, requires `cli` feature)
- `tests/` - Integration tests
- `nix/modules/` - Modular Nix configuration
  - `devshell.nix` - Development shell configuration
  - `rust.nix` - Rust build configuration via rust-flake/crane
  - `pre-commit.nix` - Code formatting and linting hooks
- `justfile` - Command shortcuts for common development tasks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orual/letta-rs](https://github.com/orual/letta-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
