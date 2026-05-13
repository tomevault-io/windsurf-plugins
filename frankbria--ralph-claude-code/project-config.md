---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Ralph for Claude Code repository - an autonomous AI development loop system that enables continuous development cycles with intelligent exit detection and rate limiting.

See [README.md](README.md) for version info, changelog, and user documentation.

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
6. **ralph_enable.sh** - Interactive wizard for enabling Ralph in existing projects
   - Multi-step wizard with environment detection, task source selection, configuration
   - Imports tasks from beads, GitHub Issues, or PRD documents
   - Generates `.ralphrc` project configuration file
7. **ralph_enable_ci.sh** - Non-interactive version for CI/automation
   - Same functionality as interactive version with CLI flags
   - JSON output mode for machine parsing
   - Exit codes: 0 (success), 1 (error), 2 (already enabled)

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
   - Extracts structured fields: status, exit_signal, work_type, files_modified, asking_questions, question_count
   - **Question detection**: `detect_questions()` with `QUESTION_PATTERNS` array — detects when Claude asks questions instead of acting autonomously (Issue #190)
   - **Session management**: `store_session_id()`, `get_last_session_id()`, `should_resume_session()`
   - Automatic session persistence to `.ralph/.claude_session_id` file with 24-hour expiration
   - Session lifecycle: `get_session_id()`, `reset_session()`, `log_session_transition()`, `init_session_tracking()`
   - Session history tracked in `.ralph/.ralph_session_history` (last 50 transitions)
   - Session auto-reset on: circuit breaker open, manual interrupt, project completion
   - Detects test-only loops, stuck error patterns, and question-only loops
   - Two-stage error filtering to eliminate false positives
   - Multi-line error matching for accurate stuck loop detection
   - **Mode-specific heuristic exit**: In JSON mode, heuristics are suppressed entirely — only an explicit `EXIT_SIGNAL: true` in a RALPH_STATUS block can set `exit_signal=true` (Issue #224). In text mode, exit requires `confidence_score >= 70` AND `has_completion_signal=true` (raised from the old `>= 40 OR has_completion_signal` to prevent documentation keywords like "setup is done" from triggering false-positive exits).

3. **lib/date_utils.sh** - Cross-platform date utilities
   - ISO timestamp generation for logging
   - Epoch time calculations for rate limiting
   - ISO-to-epoch conversion for cooldown timer comparisons (`parse_iso_to_epoch()`)

4. **lib/timeout_utils.sh** - Cross-platform timeout command utilities
   - Detects and uses appropriate timeout command for the platform
   - Linux: Uses standard `timeout` from GNU coreutils
   - macOS: Uses `gtimeout` from Homebrew coreutils
   - `portable_timeout()` function for seamless cross-platform execution
   - Automatic detection with caching for performance

5. **lib/enable_core.sh** - Shared logic for ralph enable commands
   - Idempotency checks: `check_existing_ralph()`, `is_ralph_enabled()`
   - Safe file operations: `safe_create_file()`, `safe_create_dir()`
   - Project detection: `detect_project_context()`, `detect_git_info()`, `detect_task_sources()`
   - Template generation: `generate_prompt_md()`, `generate_agent_md()`, `generate_fix_plan_md()`, `generate_ralphrc()`

6. **lib/wizard_utils.sh** - Interactive prompt utilities for enable wizard
   - User prompts: `confirm()`, `prompt_text()`, `prompt_number()`
   - Selection utilities: `select_option()`, `select_multiple()`, `select_with_default()`
   - Output formatting: `print_header()`, `print_bullet()`, `print_success/warning/error/info()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
