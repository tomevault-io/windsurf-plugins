---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

zcmds is a cross-platform productivity command-line toolset written in Python. It provides utilities for media manipulation through ffmpeg, AI integration, and common Unix commands on Windows. The project is actively maintained and uses modern Python packaging standards.

## Development Commands

### Essential Commands

```bash
# Linting (MUST run before committing)
./lint  # Runs isort, black, ruff, and mypy

# Testing (MUST run before committing)
./test  # Runs unittest discovery

# Build and Install
./install  # Installs package in development mode
./clean    # Cleans build artifacts

# Package Management
pip install -e .  # Install in development mode
uv run python setup.py sdist bdist_wheel  # Build distribution
./upload_package.sh  # Upload to PyPI

# Python Execution
# IMPORTANT: Always use uv to run Python commands
uv run python script.py  # Use this instead of python script.py
```

### Code Quality Requirements

Before marking any task as complete, you MUST:
1. Run `./lint` and ensure it passes with no errors
2. Run `./test` and ensure all tests pass

Both commands must exit with status code 0.

## Architecture

### Project Structure

- **src/zcmds/**: Main package directory
  - **cmds/**: Command implementations organized by platform
    - **common/**: Cross-platform commands (majority of tools)
    - **darwin/**: macOS-specific commands
    - **linux/**: Linux-specific commands
  - **util/**: Shared utility functions
  - **install/**: Installation-related code
  - **cmds.txt**: Registry of all available commands (entry points)

### Command Registration

New commands are registered in `src/zcmds/cmds.txt` using the format:
```
command_name = "zcmds.cmds.common.module_name:main"
```

The `setup.py` dynamically reads this file to generate console script entry points.

### Key Design Patterns

1. **Platform-specific implementations**: Commands check `sys.platform` and import platform-specific utilities when needed (e.g., `zcmds_win32` for Windows)

2. **FFmpeg integration**: Many video/audio commands use `static-ffmpeg` for cross-platform media processing

3. **AI tool integration**: Commands like `askai` and `aicode` integrate with external AI packages (`advanced-askai`, `advanced-aicode`)

4. **Command structure**: Each command module typically has a `main()` function that serves as the entry point

## How to Add a New Command

When Claude needs to add a new command to the zcmds project, follow this systematic approach:

### Step 1: Create the Command Module

Create a new Python file in `src/zcmds/cmds/common/` with the command name:

```python
# src/zcmds/cmds/common/commandname.py
import subprocess
import sys


def main() -> int:
    """
    Brief description of what this command does.
    This function serves as the entry point for the command.
    """
    try:
        # Command implementation here
        print("Command executed successfully!")
        return 0

    except FileNotFoundError as e:
        print(f"Error: Required tool not found - {e}", file=sys.stderr)
        return 1
    except KeyboardInterrupt:
        print("\nOperation cancelled by user", file=sys.stderr)
        return 130
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        return 1


if __name__ == "__main__":
    sys.exit(main())
```

### Step 2: Register the Command

Add the command to `src/zcmds/cmds.txt` in alphabetical order:

```
commandname = "zcmds.cmds.common.commandname:main"
```

**Critical**: You must include ALL existing commands in `cmds.txt`. The file should contain the complete list of commands, not just the new one. Use the Glob tool to find all Python files in `src/zcmds/cmds/common/` and generate the complete list.

### Step 3: Follow Code Quality Standards

Before marking the task complete, you MUST:

1. **Run linting**: `./lint` must pass with no errors
2. **Run tests**: `./test` must pass with all tests successful
3. **Reinstall package**: `./install` to register the new command
4. **Verify registration**: Check that the command appears in `zcmds` output

### Step 4: Implementation Guidelines

- **Return codes**: Use 0 for success, non-zero for errors (1 for general errors, 130 for KeyboardInterrupt)
- **Error handling**: Always use comprehensive try-except blocks and print errors to stderr
- **Exception handling**: MUST implement proper exception handling for all functions:
  - Log all exceptions using the `logging` module
  - Handle `KeyboardInterrupt` by calling `_thread.interrupt_main()` and logging the interruption
  - Use structured exception handling with specific exception types
  - **CRITICAL**: Always include general `Exception` catch-all with logging - you MUST log every exception that occurs
  - Never ignore exceptions - every exception must be logged before handling
- **Logging**: Configure logging with both file and stderr output for error tracking
- **Documentation**: Include clear docstrings explaining the command's purpose
- **Arguments**: Handle command-line arguments appropriately (consider using `argparse` for complex cases)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zackees/zcmds](https://github.com/zackees/zcmds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
