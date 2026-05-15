---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

cchook is a CLI tool that simplifies Claude Code hook configuration by providing YAML-based configuration and template syntax instead of complex JSON one-liners. It processes JSON input from Claude Code hooks via stdin and executes configured actions based on matching conditions.

## Example Configuration

```yaml
# Prevent building when build directory already exists
PreToolUse:
  - matcher: "Bash"
    conditions:
      - type: dir_exists
        value: "build"
      - type: command_starts_with
        value: "make"
    actions:
      - type: output
        message: "Build directory already exists. Run 'make clean' first."
        exit_status: 1

# Warn when package-lock.json doesn't exist
PreToolUse:
  - matcher: "Bash"
    conditions:
      - type: file_not_exists
        value: "package-lock.json"
      - type: command_starts_with
        value: "npm install"
    actions:
      - type: output
        message: "Warning: package-lock.json not found. This may cause dependency issues."

# Create backup directory if it doesn't exist
PreToolUse:
  - matcher: "Write|Edit"
    conditions:
      - type: dir_not_exists
        value: "backups"
    actions:
      - type: command
        command: "mkdir -p backups && echo 'Created backup directory'"

# Check for missing test files
PostToolUse:
  - matcher: "Write"
    conditions:
      - type: file_extension
        value: ".go"
      - type: file_not_exists_recursive
        value: "main_test.go"
    actions:
      - type: output
        message: "Consider adding tests for {.tool_input.file_path}"

# Pass complex JSON data to external commands via stdin
PreToolUse:
  - matcher: "Write|Edit"
    conditions:
      - type: file_extension
        value: ".sql"
    actions:
      - type: command
        # use_stdin: true safely handles special characters (newlines, quotes, etc.)
        # without shell escaping issues
        command: "python validate_sql.py"
        use_stdin: true
```

## Development Commands

```bash
# Build the project
go build -o cchook

# Install dependencies
go mod download
go mod tidy

# Run unit tests only (fast, no external dependencies)
go test ./...

# Run unit tests with verbose output
go test -v ./...

# Run integration tests (requires external commands like cat, jq)
go test -tags=integration ./...

# Run integration tests with verbose output
go test -v -tags=integration ./...

# Run specific test function (more practical than test file)
go test -v -run TestCheckGitTrackedFileOperation ./...
go test -v -run TestExecutePreToolUseHooks ./...
go test -v -run TestCheckUserPromptSubmitCondition ./...

# Run specific integration test
go test -v -tags=integration -run TestExecutePreToolUseAction_WithUseStdin ./...

# Run with coverage
go test -cover ./...

# Run with coverage report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out

# Install locally for testing
go install

# Lint code (via pre-commit) - requires pre-commit to be installed
pre-commit run --all-files

# Lint Go code directly with golangci-lint
golangci-lint run

# Test the tool manually (requires JSON input via stdin)
echo '{"session_id":"test","hook_event_name":"PreToolUse","tool_name":"Write","tool_input":{"file_path":"test.go"}}' | ./cchook -event PreToolUse

# Dry-run mode for testing configurations
echo '{"session_id":"test","hook_event_name":"PreToolUse","tool_name":"Write","tool_input":{"file_path":"test.go"}}' | ./cchook -event PreToolUse -command "echo 'would execute: {.tool_name}'"
```

## Architecture

### Core Components

The application follows a modular architecture with clear separation of concerns:

**Main Entry Point** (`main.go`)
- CLI argument parsing (`-config`, `-command`, `-event`)
- Delegates to appropriate hook execution functions
- Handles ExitError for proper exit codes and output routing

**Type System** (`types.go`)
- Defines all event types: PreToolUse, PostToolUse, Stop, SubagentStop, Notification, PreCompact, SessionStart, SessionEnd, UserPromptSubmit
- Event-specific input structures with embedded BaseInput
- Hook and Action interfaces for polymorphic behavior
- Separate condition and action types for each event
- Opaque struct pattern for ConditionType with predefined singletons

**Configuration** (`config.go`)
- YAML configuration loading with XDG_CONFIG_HOME support
- Default path: `~/.config/cchook/config.yaml`
- Custom path via `-config` flag

**Input Processing** (`parser.go`)
- Generic parsing function with type constraints
- Tool-specific parsing for PreToolUse/PostToolUse (complex tool_input handling)
- Returns both structured data and raw JSON for template processing

**Hook Execution**
- `hooks_dispatch.go`: Entry points and routing for all hook types
- `hooks_execute.go`: Standard event hook execution (8 events: Notification, SubagentStart, Stop, SubagentStop, PreCompact, SessionStart, UserPromptSubmit, SessionEnd)
- `hooks_tool_permission.go`: Tool and permission related hooks (PreToolUse, PostToolUse, PermissionRequest)
- `hooks_dryrun.go`: Dry-run mode implementations for all 11 event types


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syou6162/cchook](https://github.com/syou6162/cchook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
