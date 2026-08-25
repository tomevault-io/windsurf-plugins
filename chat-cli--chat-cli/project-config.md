---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## AI-DLC

When the user invokes AI-DLC, read and follow
`.aidlc/aidlc-rules/aws-aidlc-rules/core-workflow.md` to start the workflow.

## Work Tracking

**Always use GitHub issues to track ideas and document work in this repo.**

- New ideas, features, and bugs get a GitHub issue before work starts, even for small items.
- Use the issue to document decisions, progress, and open questions as the work happens, not just at the end.
- Link pull requests to the issue they address (e.g. `Closes #123`) so the issue reflects the outcome.
- If work surfaces a new idea or follow-up that's out of scope for the current task, open a new issue for it rather than letting it get lost.

## Project Overview

Chat-CLI is a terminal-based program built with Go and Cobra that provides an interface to Amazon Bedrock LLMs. It supports interactive chat sessions, one-shot prompts, image generation, and persistent configuration management.

**Key Technologies:**
- Go 1.24+
- Cobra (CLI framework)
- Viper (configuration management)
- SQLite (chat history persistence)
- AWS SDK v2 (Bedrock integration)
- Charm libraries (TUI components: bubbletea, bubbles, lipgloss)

## Architecture Principles

### Clean Separation of Concerns

The codebase follows a layered architecture:

1. **CLI Layer** (`/cmd/`) - Cobra commands and user interaction
2. **Configuration Layer** (`/config/`) - OS-specific config management with Viper
3. **Database Layer** (`/db/` and `/repository/`) - SQLite persistence with repository pattern
4. **Factory Layer** (`/factory/`) - Database factory pattern
5. **Utils Layer** (`/utils/`) - Utility functions for document loading

### Key Design Patterns

**Root Command Behavior:**
- Running `chat-cli` without arguments launches interactive chat directly
- All chat flags (--model-id, --custom-arn, --chat-id) work at root level
- Subcommands provide additional functionality (prompt, config, image, models)

**Configuration Precedence:**
1. CLI flags (highest priority)
2. Config file values (set via `config set`)
3. Built-in defaults (lowest priority)

**Model Selection Priority:**
1. `--custom-arn` flag (highest)
2. `--model-id` flag
3. Config file values
4. Default: `us.anthropic.claude-sonnet-5`

## Development Workflow

### Building and Running

```bash
# Build the CLI binary
make

# Run from build directory
./bin/chat-cli <command> <args> <flags>

# Run directly with Go (development)
go run main.go <command> <args> <flags>
```

### Testing Requirements

**CRITICAL: Always follow Test-Driven Development (TDD)**

Before implementing any feature:
1. Write tests first that define the expected behavior
2. Run the test and watch it fail for the right reason
3. Write minimal code to make the test pass
4. Run the full test suite to verify nothing else broke
5. Refactor while keeping tests green

**Test Commands:**
```bash
# Run all tests
make test

# Run tests in short mode (skip integration tests)
make test-short

# Run tests with coverage
make test-coverage

# Run integration tests (requires built CLI)
make cli && go test -tags=integration -v

# Run linting
make lint
```

**Test Guidelines:**
- Keep tests focused and test one thing at a time
- Use descriptive test names that explain what is being tested
- Follow Go testing conventions and use the `testing` package
- Place tests in `*_test.go` files alongside the code they test

**Coverage Goals:**
- New functions: 80%+ coverage
- Critical paths: 90%+ coverage
- Test both success and failure scenarios
- Include edge cases and boundary conditions

**Current Coverage:**
- Repository: 80.6%
- Config: 77.2%
- Utils: 46.6%
- CMD: 7.4%

### Code Quality Standards

**Before Committing:**
```bash
make test                    # Ensure all tests pass
make test-coverage          # Verify coverage hasn't decreased
make lint                   # Check code quality
make cli && go test -tags=integration -v  # Test CLI integration
```

## File Organization

### Core Files
- `main.go` - Entry point, delegates to cmd package
- `go.mod` / `go.sum` - Go module dependencies
- `Makefile` - Build and test automation
- `integration_test.go` - Integration tests

### Directory Structure
- `/cmd/` - All Cobra command implementations
- `/config/` - Configuration management
- `/db/` - Database layer and migrations
- `/repository/` - Repository pattern implementations
- `/factory/` - Factory pattern for database
- `/utils/` - Utility functions
- `/docs/` - Sphinx documentation (Python-based)
- `/bin/` - Compiled binaries (gitignored)
- `/dist/` - Release artifacts (gitignored)

### Command Files
- `root.go` - Base command, launches chat when no subcommands
- `chat.go` - Interactive chat sessions with persistence
- `chatList.go` - List recent chat sessions
- `prompt.go` - One-shot prompts with stdin support
- `config.go` - Configuration management (set/get/unset)
- `image.go` - Image generation commands
- `models.go` / `modelsList.go` - Model listing and management
- `version.go` - Version information

## Key Features and Implementation Details

### Chat Session Management
- Auto-saves all interactions to SQLite
- Sessions identified by UUID

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chat-cli/chat-cli](https://github.com/chat-cli/chat-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
