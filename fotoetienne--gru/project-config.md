---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gru is a local-first LLM agent orchestrator that autonomously works on GitHub issues using Claude Code. It manages "Minions" (agent sessions) that claim issues, implement fixes, create PRs, and respond to reviews.

**Current Phase:** V1 - Basic delegation to Claude Code CLI
**Language:** Rust
**Architecture:** Single binary, GitHub as state store, git worktrees for isolation

## Build & Development Commands

```bash
# Build
just build

# Build with release optimizations
just build-release

# Run tests
just test

# Run tests with output
just test-verbose

# Run linter (with warnings as errors)
just lint

# Automatically fix clippy lints where possible
just fix-clippy

# Format code
just fmt

# Check formatting without modifying
just fmt-check

# Run all checks (format + lint + test + build)
just check

# Install locally
just install

# Clean build artifacts
just clean

# Show project information
just info
```

## Pre-commit Hooks

The project uses git hooks from `.githooks/` that automatically run:
- Code formatting check (`just fmt-check`)
- Linting (`just lint`)
- Tests (`just test`)
- Branch protection (blocks commits to main)
- TODO/FIXME warnings (non-blocking)

**To enable hooks:**
```bash
git config core.hooksPath .githooks
```

**When pre-commit hooks fail:**
1. Fix the issues using the suggested commands (e.g., `just fmt` or `just fix-clippy`)
2. **Review the diff** with `git diff` to see what changed
3. **Stage the fixed files selectively** with `git add <files>`
4. Never use `git add .` or `git add -A` - always stage files explicitly
5. Example workflow:
   ```bash
   git commit -m "Fix bug"
   # Pre-commit hook fails with formatting errors
   just fmt
   git diff              # Review what changed
   git add src/main.rs   # Stage only the files you want to commit
   git commit -m "Fix bug"
   ```

**To bypass in emergencies:**
```bash
git commit --no-verify
```

## Code Architecture

### Module Structure

- `src/main.rs` - CLI entry point using Clap, defines commands: init, do, review, rebase, path, attach, resume, clean, status, stop, prompt, prompts, lab, chat, logs, tail
- `src/commands/` - Command handlers
  - `fix/` - Handles `gru do`: directory module with submodules for worktree setup, agent execution, monitoring, PR creation, and resolution
  - `review.rs` - Delegates PR review to Claude CLI
  - `prompt.rs` - Ad-hoc prompt execution (literal or from prompt files)
  - `prompts.rs` - Lists available prompt files
  - `status.rs` - Lists active Minions from registry
  - `clean.rs` - Removes merged/closed worktrees
  - `path.rs` - Resolves Minion worktree paths
  - `attach.rs` - Attach terminal to a running Minion
  - `resume.rs` - Resume a stopped Minion
  - `stop.rs` - Stop a running Minion
  - `rebase.rs` - Rebase a Minion branch onto base
  - `lab.rs` - Daemon mode (polls for issues, spawns Minions)
  - `init.rs` - Initialize workspace for a repo
  - `chat.rs` - Interactive chat with a Minion
  - `logs.rs` - View Minion log files
  - `tail.rs` - Tail Minion event streams
  - `child_process.rs` - Child process management utilities
- `src/agent.rs` - AgentBackend trait and AgentEvent model for multi-agent support
- `src/agent_registry.rs` - Backend registration and resolution (maps agent names to backends)
- `src/agent_runner.rs` - Generic agent execution and monitoring (backend-agnostic)
- `src/claude_backend.rs` - Claude Code CLI backend implementation (implements AgentBackend)
- `src/codex_backend.rs` - OpenAI Codex CLI backend implementation (implements AgentBackend)
- `src/minion.rs` - Minion ID generation (monotonic counter, base36 format: M000, M001, etc.)
- `src/minion_registry.rs` - Persistent Minion tracking (`~/.gru/state/minions.json` with file locking)
- `src/minion_resolver.rs` - Resolve Minion by ID, issue number, or PR number
- `src/workspace.rs` - Manages `~/.gru/` directory structure (repos, work, archive, state)
- `src/git.rs` - Git operations (bare repos, worktrees, branch management)
- `src/claude_runner.rs` - Claude CLI command builders (constructs TokioCommand values for ClaudeBackend)
- `src/stream.rs` - Claude Code JSON stream parser (events: message_start, content_block_delta, etc.)
- `src/progress.rs` - Terminal progress display (spinner, tool status)
- `src/ci.rs` - CI monitoring, failure analysis, and auto-fix via Claude
- `src/pr_monitor.rs` - PR polling for reviews, CI status, and merge state
- `src/pr_state.rs` - PR state persistence (`.gru_pr_state.json`)
- `src/worktree_scanner.rs` - Discovers and checks status of worktrees
- `src/github.rs` - GitHub API functions (gh CLI wrappers)
- `src/url_utils.rs` - Issue/PR URL parsing
- `src/config.rs` - TOML configuration loader (`~/.gru/config.toml`)
- `src/prompt_loader.rs` - Built-in and custom prompt file loading
- `src/prompt_renderer.rs` - `{{ variable }}` template rendering for prompts
- `src/text_buffer.rs` - Streaming text buffer with flush intervals
- `src/progress_comments.rs` - GitHub PR progress comment formatting
- `src/labels.rs` - GitHub label constants and helpers (`gru:todo`, `gru:in-progress`, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fotoetienne/gru](https://github.com/fotoetienne/gru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
