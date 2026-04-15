---
trigger: always_on
description: The pedalboard-pluginary project has successfully implemented a robust, failsafe scanning architecture with complete process isolation, journaling capabilities, and SQLite-based storage. The system reliably scans audio plugins (VST3, AU) with crash recovery, resume functionality, and efficient data storage.
---

# Pedalboard Pluginary

## 1. Overview

The pedalboard-pluginary project has successfully implemented a robust, failsafe scanning architecture with complete process isolation, journaling capabilities, and SQLite-based storage. The system reliably scans audio plugins (VST3, AU) with crash recovery, resume functionality, and efficient data storage.

## 2. Current Architecture (Completed)

### 2.1. Core System Components
- **Process Isolation**: Each plugin scanned in separate subprocess via `scan_single.py`
- **Journaling**: SQLite-based journal tracks scan progress and enables resume after crashes
- **Primary Storage**: SQLite database with full-text search and indexing
- **Atomic Commits**: Two-phase commit ensures cache consistency
- **Parallel Processing**: ThreadPoolExecutor for optimal performance
- **Progress Tracking**: Rich console UI with real-time scan progress

### 2.2. Key Modules
- `scanner_isolated.py`: Main scanner orchestrator with ScanJournal integration
- `scan_single.py`: Standalone CLI for scanning individual plugins
- `cache/sqlite_backend.py`: SQLite cache backend with FTS5 search
- Rich CLI interface with commands: scan, update, list, info, json, yaml

### 2.3. Recent Achievements (2025-08-06)
- Successfully migrated to SQLite as sole storage backend
- Fixed all critical scanner and journal issues
- Achieved successful scanning of 289 plugins
- JSON/YAML export working from SQLite
- Eliminated dependency on JSON file storage


# Software Development Rules

## 3. Pre-Work Preparation

### 3.1. Before Starting Any Work
- **ALWAYS** read `WORK.md` in the main project folder for work progress
- Read `README.md` to understand the project
- STEP BACK and THINK HEAVILY STEP BY STEP about the task
- Consider alternatives and carefully choose the best option
- Check for existing solutions in the codebase before starting

### 3.2. Project Documentation to Maintain
- `README.md` - purpose and functionality
- `CHANGELOG.md` - past change release notes (accumulative)
- `PLAN.md` - detailed future goals, clear plan that discusses specifics
- `TODO.md` - flat simplified itemized `- [ ]`-prefixed representation of `PLAN.md`
- `WORK.md` - work progress updates

## 4. General Coding Principles

### 4.1. Core Development Approach
- Iterate gradually, avoiding major changes
- Focus on minimal viable increments and ship early
- Minimize confirmations and checks
- Preserve existing code/structure unless necessary
- Check often the coherence of the code you're writing with the rest of the code
- Analyze code line-by-line

### 4.2. Code Quality Standards
- Use constants over magic numbers
- Write explanatory docstrings/comments that explain what and WHY
- Explain where and how the code is used/referred to elsewhere
- Handle failures gracefully with retries, fallbacks, user guidance
- Address edge cases, validate assumptions, catch errors early
- Let the computer do the work, minimize user decisions
- Reduce cognitive load, beautify code
- Modularize repeated logic into concise, single-purpose functions
- Favor flat over nested structures

## 5. Tool Usage (When Available)

### 5.1. Additional Tools
- If we need a new Python project, run `curl -LsSf https://astral.sh/uv/install.sh | sh; uv venv --python 3.12; uv init; uv add fire rich; uv sync`
- Use `tree` CLI app if available to verify file locations
- Check existing code with `.venv` folder to scan and consult dependency source code
- Run `DIR="."; uvx codetoprompt --compress --output "$DIR/llms.txt"  --respect-gitignore --cxml --exclude "*.svg,.specstory,*.md,*.txt,ref,testdata,*.lock,*.svg" "$DIR"` to get a condensed snapshot of the codebase into `llms.txt`

## 6. File Management

### 6.1. File Path Tracking
- **MANDATORY**: In every source file, maintain a `this_file` record showing the path relative to project root
- Place `this_file` record near the top:
- As a comment after shebangs in code files
- In YAML frontmatter for Markdown files
- Update paths when moving files
- Omit leading `./`
- Check `this_file` to confirm you're editing the right file

## 7. Python-Specific Guidelines

### 7.1. PEP Standards
- PEP 8: Use consistent formatting and naming, clear descriptive names
- PEP 20: Keep code simple and explicit, prioritize readability over cleverness
- PEP 257: Write clear, imperative docstrings
- Use type hints in their simplest form (list, dict, | for unions)

### 7.2. Modern Python Practices
- Use f-strings and structural pattern matching where appropriate
- Write modern code with `pathlib`
- ALWAYS add "verbose" mode loguru-based logging & debug-log
- Use `uv add` 
- Use `uv pip install` instead of `pip install`
- Prefix Python CLI tools with `python -m` (e.g., `python -m pytest`)

### 7.3. CLI Scripts Setup
For CLI Python scripts, use `fire` & `rich`, and start with:
```python
#!/usr/bin/env -S uv run -s
# /// script
# dependencies = ["PKG1", "PKG2"]
# ///
# this_file: PATH_TO_CURRENT_FILE
```

### 7.4. Post-Edit Python Commands
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twardoch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
