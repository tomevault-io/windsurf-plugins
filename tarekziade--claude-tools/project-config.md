---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**claude-tools** is a Python package providing hooks and utilities for Claude Code. The primary tool is a traceback compactor that intelligently reduces Python stack traces to save tokens while preserving debugging information.

**Core Design Principles:**
- Zero runtime dependencies (pure Python stdlib)
- Fast execution (runs as CLI hook with timeouts)
- Python 3.8+ compatibility
- Deterministic and testable

## Quick Start Commands

```bash
# Development setup
make install-dev        # Install with dev dependencies (ruff, coverage)
make verify-install     # Verify installation works

# Testing
make test              # Run 32-test suite
make test-verbose      # Verbose test output
python3 -m unittest tests.test_trace_compactor.TestParseTraceback -v  # Run specific test class

# Code quality
make format            # Auto-format with ruff
make lint              # Check code quality
make dev-check         # Format + lint + test (run before committing)

# Build & publish
make clean             # Remove build artifacts
make build             # Build distribution packages
make publish-test      # Upload to TestPyPI
make publish           # Upload to PyPI

# View all commands
make help
```

## Architecture

### High-Level Design

The project implements a **hook-based traceback compaction system** that intercepts text at two points:

1. **User input** (UserPromptSubmit hook) - Compacts tracebacks you paste
2. **Tool output** (PostToolUse hook) - Compacts tracebacks from Python scripts Claude runs

**Data flow:**
```
Text with traceback
    ↓
Regex detection (TRACEBACK_BLOCK_RE)
    ↓
Parse frames (FRAME_RE, CODE_RE, EXC_LINE_RE)
    ↓
Score frames (project_root bonus: +100, user code: +10, recency: -index)
    ↓
Select top N frames (default: 4)
    ↓
Generate compact summary with fingerprint
    ↓
Replace verbose block with <COMPACT_PY_TRACEBACK>
```

### Key Components

**1. Core Module** (`ctools/trace_compactor.py`)

Main functions:
- `parse_traceback_text(text)` - Extract frames and exception from traceback text
- `compact_traceback_block(parsed, max_frames, project_root)` - Score and select relevant frames
- `rewrite_prompt_for_claude(prompt, project_root, max_frames)` - Main entry point, replaces all tracebacks
- `_frame_score(frame, project_root)` - Scoring algorithm for frame relevance
- `_fingerprint(frames, exception)` - Deterministic hash for deduplication

**2. Hook Script** (`.claude/hooks/compact-traceback.sh`)

Unified script handling both hook types:
- Detects hook type via `hook_event_name` field
- Routes to appropriate processing (prompt vs tool output)
- Calls `claude-trace-compactor` CLI with `--stdin`
- Returns JSON in correct format for each hook type

**3. CLI** (`claude-trace-compactor` command)

Entry point: `_cli_main()` in `trace_compactor.py`
- Registered in `pyproject.toml` under `[project.scripts]`
- Options: `--stdin`, `--file`, `--project-root`, `--max-frames`, `--json`

### Frame Scoring Algorithm

Critical to understand for modifications:

```python
def _frame_score(frame, project_root):
    score = -frame['raw_index']  # Recency: recent frames score higher

    if project_root and frame['filename'].startswith(project_root):
        score += 100  # Project frames highest priority

    if not _is_stdlib_or_site_packages(frame['filename']):
        score += 10   # User code over library code

    return score
```

Frames are sorted by score (descending), top N selected, then re-sorted by `raw_index` to preserve chronological order.

### Regex Patterns

**TRACEBACK_BLOCK_RE** - Matches complete traceback blocks:
- Starts with `Traceback (most recent call last):`
- Captures one or more frame lines (`File "...", line N, in func`)
- Optional code line after each frame
- Ends with exception line (`Error|Exception|Warning...`)

**FRAME_RE** - Extracts frame components:
- Captures: filename, line number, function name

**EXC_LINE_RE** - Extracts exception details:
- Captures: exception type, message

### Test Structure

```
tests/
├── test_trace_compactor.py     # Core functionality (6 test classes)
│   ├── TestParseTraceback      # Parsing logic
│   ├── TestCompactTraceback    # Compaction logic
│   ├── TestRewritePrompt       # Full pipeline
│   ├── TestFrameScoring        # Scoring algorithm
│   ├── TestEdgeCases           # Malformed input, unicode
│   └── TestFingerprinting      # Deduplication
└── test_cli.py                 # CLI tests (2 test classes)
    ├── TestCLI                 # Basic CLI operations
    └── TestCLIIntegration      # Real-world scenarios
```

**32 total tests** - All use stdlib `unittest`, zero test dependencies.

## Development Workflow

### Pre-commit Checklist

**Always run before committing:**
```bash
make dev-check   # Formats, lints, and tests
```

This runs:
1. `ruff format` - Auto-format code
2. `ruff check` - Lint with auto-fix
3. `python3 -m unittest` - Run all tests

### Adding New Tools

When adding additional tools to `ctools/`:

1. **Create module** in `ctools/` (e.g., `ctools/log_summarizer.py`)
2. **Export main functions** in `ctools/__init__.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarekziade/claude-tools](https://github.com/tarekziade/claude-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
