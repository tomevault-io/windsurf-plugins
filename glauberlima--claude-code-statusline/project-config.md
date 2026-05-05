---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bash-based statusline for Claude Code CLI displaying (in order):

- Directory (📁)
- Git branch (🌿) when in a Git repository
- File changes (✏️) when present
- Model name (🤖)
- Context usage visualization with progress bar and funny messages (📊)
- Cost tracking (💰) when present

**Primary file**: `statusline.sh`
**Language**: Bash 3.2+ with POSIX compatibility considerations

## Development Commands

### Testing

```bash
# Run all tests
./tests/unit.sh && ./tests/integration.sh && ./tests/shellcheck.sh

# Individual test suites
./tests/unit.sh          # Component-level tests (< 1s)
./tests/integration.sh   # End-to-end tests with JSON fixtures
./tests/shellcheck.sh    # Bash syntax (bash -n) + static analysis (zero-tolerance, all checks enabled)

# Manual testing
cat tests/fixtures/test-input.json | ./statusline.sh
```

### Installation

```bash
./install.sh  # Copies statusline.sh to ~/.claude/statusline.sh (always copies, no symlink mode)
```

### Linting

```bash
# Automated (recommended - includes bash -n syntax check)
./tests/shellcheck.sh

# Manual shellcheck only
shellcheck statusline.sh install.sh tests/*.sh  # Uses .shellcheckrc config
```

## Architecture

### Component-Based Flow

```
JSON Input → Parse (awk) → Build Components → Assemble → ANSI Output
```

**Key functional areas in statusline.sh**:

- **Configuration**: Colors (ANSI codes), icons (emoji), constants (bar width, separators)
- **i18n**: Messages statically compiled via `@MESSAGES_START` / `@MESSAGES_END` markers
- **Utilities**: Directory name extraction (`get_dirname`), separator formatting (`sep`), path validation (`validate_directory`), pipe-field parsing (`read_pipe_fields`)
- **Core logic**: JSON parsing (`parse_claude_input`), git operations (`get_git_info`), progress bar rendering (`build_progress_bar`)
- **Formatters**: Transform raw data to display format (`format_ahead_behind`, `format_git_branch`)
- **Component builders**: Individual statusline segments (`build_model_component`, `build_context_component`, `build_directory_component`, `build_git_component`, `build_files_component`, `build_cost_component`)
- **Assembly**: Combine components with separators (`assemble_statusline`)
- **Orchestration**: Main entry point and dependency checks (`main`)

### Design Patterns Applied

- **Single Responsibility**: Each function has one purpose (parse, format, build, assemble)
- **Open/Closed**: Add components without modifying existing code
- **DRY**: Reusable helpers (`is_present()`, `format_ahead_behind()`, `sep()`)
- **Functional Composition**: Functions pipe data through transformation stages

### Critical Performance Optimization

**Git operations: 1 call for all status data**:

- `git status --porcelain=v2 --branch --untracked-files=all` - Provides branch, upstream, ahead/behind, file status in single call

This porcelain v2 format requires **git 2.11+** (Dec 2016).

**Why**: Reduces subprocess overhead by ~85% compared to naive approach (7 separate git calls).

### Security Features

**Path validation**:

- `validate_directory()`: Prevents path traversal attacks, format string injection
- Validates against patterns: `..`, format specifiers, null bytes
- **Allows absolute paths** — absolute paths are valid; only `..` traversal, format specifiers, and null bytes are rejected
- All user-controlled inputs (workspace.current_dir) validated before use

**Input sanitization**: `workspace.current_dir` from JSON is validated before any shell operations.

### State Management

**Git state constants**:

- `STATE_NOT_REPO`: Not a git repository
- `STATE_CLEAN`: No modified files
- `STATE_DIRTY`: Has modified files

## Internationalization (i18n)

### Architecture

The statusline uses a **static patching system** for zero-runtime overhead:

```
statusline.sh (English default)
        ↓
patch-statusline.sh + messages/pt.json
        ↓
statusline.sh (Portuguese, fully static)
```

**Key points**:
- Messages hardcoded in `statusline.sh` via `@MESSAGES_START` / `@MESSAGES_END` markers
- Configuration flags (`SHOW_MESSAGES`, `SHOW_COST`) hardcoded via `@CONFIG_START` / `@CONFIG_END` markers
- `patch-statusline.sh` script replaces marker blocks to create optimized versions
- **Zero runtime overhead** - no config loading, no JSON parsing during execution
- **Both flags default to `false`** in the base script. The installer (or `patch-statusline.sh`) enables them during setup.

### Language Files Structure

Each language file (`messages/{lang}.json`) uses a simplified format:

```json
{
  "very_low": ["message1", "message2", ...],
  "low": ["message1", "message2", ...],
  "medium": ["message1", "message2", ...],
  "high": ["message1", "message2", ...],
  "critical": ["message1", "message2", ...]
}
```

**Message Counts**:
- `very_low`: 0-20% context usage (~22 messages)
- `low`: 21-40% context usage (~22 messages)
- `medium`: 41-60% context usage (~23 messages)
- `high`: 61-80% context usage (~24 messages)
- `critical`: 81-100% context usage (~28 messages)

**Supported Languages**:
- English (en) - Default

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glauberlima/claude-code-statusline](https://github.com/glauberlima/claude-code-statusline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
