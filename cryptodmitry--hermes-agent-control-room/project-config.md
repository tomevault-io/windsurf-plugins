---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Ralph for Claude Code repository - an autonomous AI development loop system that enables continuous development cycles with intelligent exit detection and rate limiting.

**Version**: v0.9.8 | **Tests**: 276 passing (100% pass rate) | **CI/CD**: GitHub Actions

## Core Architecture

The system consists of four main bash scripts and a modular library system:

### Main Scripts

1. **ralph_loop.sh** - The main autonomous loop that executes Claude Code repeatedly
2. **ralph_monitor.sh** - Live monitoring dashboard for tracking loop status
3. **setup.sh** - Project initialization script for new Ralph projects
4. **create_files.sh** - Bootstrap script that creates the entire Ralph system
5. **ralph_import.sh** - PRD/specification import tool that converts documents to Ralph format
   - Uses modern Claude Code CLI with `--output-format json` for structured responses
   - Implements `detect_response_format()` and `parse_conversion_response()` for JSON parsing
   - Backward compatible with older CLI versions (automatic text fallback)

### Library Components (lib/)

The system uses a modular architecture with reusable components in the `lib/` directory:

1. **lib/circuit_breaker.sh** - Circuit breaker pattern implementation
   - Prevents runaway loops by detecting stagnation
   - Three states: CLOSED (normal), HALF_OPEN (monitoring), OPEN (halted)
   - Configurable thresholds for no-progress and error detection
   - Automatic state transitions and recovery

2. **lib/response_analyzer.sh** - Intelligent response analysis
   - Analyzes Claude Code output for completion signals
   - **JSON output format detection and parsing** (with text fallback)
   - Supports both flat JSON format and Claude CLI format (`result`, `sessionId`, `metadata`)
   - Extracts structured fields: status, exit_signal, work_type, files_modified
   - **Session management**: `store_session_id()`, `get_last_session_id()`, `should_resume_session()`
   - Automatic session persistence to `.claude_session_id` file with 24-hour expiration
   - Session lifecycle: `get_session_id()`, `reset_session()`, `log_session_transition()`, `init_session_tracking()`
   - Session history tracked in `.ralph_session_history` (last 50 transitions)
   - Session auto-reset on: circuit breaker open, manual interrupt, project completion
   - Detects test-only loops and stuck error patterns
   - Two-stage error filtering to eliminate false positives
   - Multi-line error matching for accurate stuck loop detection
   - Confidence scoring for exit decisions

3. **lib/date_utils.sh** - Cross-platform date utilities
   - ISO timestamp generation for logging
   - Epoch time calculations for rate limiting

## Key Commands

### Installation
```bash
# Install Ralph globally (run once)
./install.sh

# Uninstall Ralph
./install.sh uninstall
```

### Setting Up a New Project
```bash
# Create a new Ralph-managed project (run from anywhere)
ralph-setup my-project-name
cd my-project-name
```

### Running the Ralph Loop
```bash
# Start with integrated tmux monitoring (recommended)
ralph --monitor

# Start without monitoring
ralph

# With custom parameters and monitoring
ralph --monitor --calls 50 --prompt my_custom_prompt.md

# Check current status
ralph --status

# Circuit breaker management
ralph --reset-circuit
ralph --circuit-status

# Session management
ralph --reset-session    # Reset session state manually
```

### Monitoring
```bash
# Integrated tmux monitoring (recommended)
ralph --monitor

# Manual monitoring in separate terminal
ralph-monitor

# tmux session management
tmux list-sessions
tmux attach -t <session-name>
```

### Running Tests
```bash
# Run all tests (165 tests)
npm test

# Run specific test suites
npm run test:unit
npm run test:integration

# Run individual test files
bats tests/unit/test_cli_parsing.bats
bats tests/unit/test_json_parsing.bats
bats tests/unit/test_cli_modern.bats
```

## Ralph Loop Configuration

The loop is controlled by several key files and environment variables:

- **PROMPT.md** - Main prompt file that drives each loop iteration
- **@fix_plan.md** - Prioritized task list that Ralph follows
- **@AGENT.md** - Build and run instructions maintained by Ralph
- **status.json** - Real-time status tracking (JSON format)
- **logs/** - Execution logs for each loop iteration

### Rate Limiting
- Default: 100 API calls per hour (configurable via `--calls` flag)
- Automatic hourly reset with countdown display
- Call tracking persists across script restarts

### Modern CLI Configuration (Phase 1.1)

Ralph uses modern Claude Code CLI flags for structured communication:

**Configuration Variables:**
```bash
CLAUDE_OUTPUT_FORMAT="json"           # Output format: json (default) or text
CLAUDE_ALLOWED_TOOLS="Write,Bash(git *),Read"  # Allowed tool permissions
CLAUDE_USE_CONTINUE=true              # Enable session continuity
CLAUDE_MIN_VERSION="2.0.76"           # Minimum Claude CLI version
```

**CLI Options:**
- `--output-format json|text` - Set Claude output format (default: json)
- `--allowed-tools "Write,Read,Bash(git *)"` - Restrict allowed tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CryptoDmitry/hermes-agent-control-room](https://github.com/CryptoDmitry/hermes-agent-control-room) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
