---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Principles

KISS (Keep It Simple, Stupid): Simplicity should be a key goal in design. Choose straightforward solutions over complex ones whenever possible. Simple solutions are easier to understand, maintain, and debug.

YAGNI (You Aren't Gonna Need It): Avoid building functionality on speculation. Implement features only when they are needed, not when you anticipate they might be useful in the future.

Dependency Inversion: High-level modules should not depend on low-level modules. Both should depend on abstractions. This principle enables flexibility and testability.

Open/Closed Principle: Software entities should be open for extension but closed for modification. Design your systems so that new functionality can be added with minimal changes to existing code.

KEEP early versions of new concepts in the concept_library/ directory very light and simple.
we add complexity step by step and only if needed.

Minimal claude code wrappers with 0 to minimal validation.

- Always remember to use uv run when we run scripts
- We never write commit messages, we have a pre-commit hook for that so always commit with git commit nothing more

## Example Claude Code Usage

```bash
# Simple prompt
claude -p "make a hello.js script that prints hello" --allowedTools "Write" "Edit"

# Using in a script
#!/usr/bin/env -S uv run --script

import subprocess

prompt = """
GIT checkout a NEW branch.

CREATE src/cc_todo/todo.py: a zero library CLI todo app with basic CRUD.

THEN GIT stage, commit and SWITCH back to main.
"""

command = ["claude", "-p", prompt, "--allowedTools", "Edit", "Bash", "Write"]

process = subprocess.run(command, check=True)

print(f"Claude process exited with output: {process.stdout}")
```

## Claude Code Tools Reference

Claude Code has access to various tools. Here are the most commonly used ones:

| Tool         | Description                                          | Permission Required |
| ------------ | ---------------------------------------------------- | ------------------- |
| Agent        | Runs a sub-agent to handle complex, multi-step tasks | No                  |
| Bash         | Executes shell commands in your environment          | Yes                 |
| Glob         | Finds files based on pattern matching                | No                  |
| Grep         | Searches for patterns in file contents               | No                  |
| LS           | Lists files and directories                          | No                  |
| Read         | Reads the contents of files                          | No                  |
| Edit         | Makes targeted edits to specific files               | Yes                 |
| MultiEdit    | Makes targeted edits to multiple files               | Yes                 |
| Write        | Creates or overwrites files                          | Yes                 |
| NotebookEdit | Modifies Jupyter notebook cells                      | Yes                 |
| NotebookRead | Reads and displays Jupyter notebook contents         | No                  |
| WebFetch     | Fetches content from a specified URL                 | Yes                 |
| WebSearch    | Searches the web for information                     | Yes                 |
| TodoRead     | Reads todo files                                     | No                  |
| TodoWrite    | Writes to todo files                                 | Yes                 |

Permission rules can be configured using /allowed-tools or in permission settings.

## Core Commands

### Environment Setup

```bash
# Create and activate virtual environment with uv
uv venv
source .venv/bin/activate  # On Unix/macOS
# .venv\Scripts\activate  # On Windows

# Install dependencies
uv sync

# Install package in development mode
uv pip install -e .
```

### Development Commands

```bash
# Run all tests
uv run pytest

# Run specific tests
uv run pytest concept_library/full_review_loop/tests/ -v

# Format code
uv run black .

# Run linter
uv run ruff check .

# Run type checker
uv run mypy .
```

### Running Core Components

#### Individual Components

```bash
# Simple Review - Code review tool for git branches/commits
uv run python concept_library/simple_review/simple_review.py <branch_name>
uv run python concept_library/simple_review/simple_review.py <branch_name> --latest-commit

# Simple Dev - Implements fixes based on review feedback
uv run python concept_library/simple_dev/simple_dev_poc.py tmp/review.md

# Simple Validator - Validates that fixes address issues
uv run python concept_library/simple_validator/simple_validator_poc.py tmp/review.md tmp/dev_report.md

# Simple PR - Creates pull requests from validated changes
uv run python concept_library/simple_pr/simple_pr_poc.py tmp/validation.md
```

#### PRP Flow

```bash
# Run a PRP in interactive mode
uv run python concept_library/cc_PRP_flow/scripts/cc_runner_simple.py --prp test --interactive

# Run a specific PRP file
uv run python concept_library/cc_PRP_flow/scripts/cc_runner_simple.py --prp-path PRPs/custom_feature.md
```

#### Standup Report Generator

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wirasm/dylan](https://github.com/Wirasm/dylan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
