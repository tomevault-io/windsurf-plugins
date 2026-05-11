---
trigger: always_on
description: This is a Unity package (`com.mxr.claude-bridge`) that enables Claude Code to control Unity Editor operations via a file-based protocol. The package has two components:
---

# Agent Guidelines for Claude Unity Bridge

## Project Overview

This is a Unity package (`com.mxr.claude-bridge`) that enables Claude Code to control Unity Editor operations via a file-based protocol. The package has two components:

1. **Unity Package** - C# code that runs in Unity Editor, polls for commands, executes them
2. **Claude Code Skill** - Python script + documentation that Claude uses to send commands

**Key Architecture Decision**: We use a deterministic Python script instead of in-context implementation because it guarantees consistent UUID generation, file handling, polling, and error handling across all Claude sessions.

## Project Structure

### Unity Package (package/)
- `Editor/` - C# command implementations for Unity
  - `ClaudeBridge.cs` - Main coordinator, command dispatcher
  - `Commands/` - Individual command implementations (ICommand interface)
  - `Models/` - Request/Response data structures
- `Documentation/` - Package documentation
- `README.md` - Package documentation and protocol specification

### Claude Code Skill (skill/)
- `scripts/cli.py` - **THE CORE** - Handles all command execution
- `SKILL.md` - Main documentation with YAML frontmatter
- `references/` - Extended documentation
  - `COMMANDS.md` - Complete command reference
  - `EXTENDING.md` - Guide for adding custom commands
- `tests/` - pytest test suite
- `TESTING.md` - Testing documentation

### Dependencies
- **Unity**: 2021.3 or later (no additional Unity dependencies)
- **Python**: 3.8+ for the skill script (3.12 supported)
- **pytest**: For testing the Python script
- **black/flake8**: For code formatting and linting
- **pre-commit**: For git hooks
- **GitHub Actions**: CI/CD for skill tests

### Setting Up Development Environment
```bash
# Install development dependencies
cd skill
pip install -r requirements-dev.txt

# Install pre-commit hooks (recommended)
pre-commit install

# Run pre-commit on all files (first time)
pre-commit run --all-files
```

## Commands You Can Use

### Testing the Python Script
```bash
# Run pytest suite
cd skill
pytest tests/test_cli.py -v

# Run with coverage
pytest tests/test_cli.py --cov=scripts --cov-report=term-missing

# Test script help
python3 scripts/cli.py --help
```

### Testing with Unity
```bash
# These require Unity Editor to be running
python3 skill/scripts/cli.py get-status
python3 skill/scripts/cli.py compile
python3 skill/scripts/cli.py run-tests --mode EditMode
python3 skill/scripts/cli.py get-console-logs --limit 10 --filter Error
python3 skill/scripts/cli.py refresh
```

### Git Workflow
```bash
# Check status
git status

# Typical commit structure for features
git add skill/scripts/*.py          # Core implementation
git commit -m "feat: Add feature"

git add skill/*.md skill/references/ README.md  # Documentation
git commit -m "docs: Add documentation"

git add skill/tests/ .github/       # Testing
git commit -m "test: Add tests and CI"
```

## Coding Style

### Python (skill/scripts/cli.py)
- PEP 8 compliant
- 100-character line limit
- Type hints where helpful
- Docstrings for all functions
- Use f-strings for formatting
- Exit codes: 0 (success), 1 (error), 2 (timeout)

### C# (Editor/)
- Follow Unity naming conventions
- Public members: `PascalCase`
- Private fields: `_camelCase` with underscore prefix
- Interfaces: `ICommand`, `ICallbacks`
- Namespaces: `MXR.ClaudeBridge.*`
- Use `[Serializable]` for data models
- Always log with `[ClaudeBridge]` prefix

### Markdown
- Use GitHub-flavored markdown
- Code blocks with language identifiers
- Table of contents for long documents
- Clear examples with actual code

## Testing Guidelines

### Python Script Tests (Critical)
- **Framework**: pytest
- **Coverage Goal**: ~95% of cli.py
- **Location**: `skill/tests/test_cli.py`
- **Run Before Commit**: Always run pytest before committing script changes
- **CI**: GitHub Actions runs on Python 3.8-3.11, Ubuntu/macOS/Windows

### Unity Package Tests
- **Framework**: Unity Test Framework (NUnit)
- **Location**: `Tests/Editor/`
- **Current Coverage**: All commands tested (~67 tests, Phases 1-4 complete)
- **Run Tests**: Open Unity Editor → Window > General > Test Runner → EditMode → Run All
- **Testing Philosophy**: Test real behavior, not mocks (see implementation plan)
  - Mock dependencies (file system, Unity APIs), test YOUR logic's response
  - Focus on state transitions, error handling, response construction
  - Avoid testing that mocks return what you set up
- **Test Infrastructure**:
  - `Tests/Editor/MXR.ClaudeBridge.Tests.Editor.asmdef` - Test assembly definition
  - `Tests/Editor/TestHelpers/CommandTestFixture.cs` - Base class for command tests
  - `Tests/Editor/TestHelpers/ResponseCapture.cs` - Utility to capture callbacks
- **Phased Rollout** (matching Python suite quality - 22 tests, 95% coverage):
  - ✅ Phase 1: Foundation + GetStatusCommand (8 tests) - COMPLETE
  - ✅ Phase 2: RefreshCommand (7 tests) + model tests (11 tests) - COMPLETE
  - ✅ Phase 3: CompileCommand async tests (9 tests) - COMPLETE
  - ✅ Phase 4: RunTestsCommand (17 tests) + GetConsoleLogsCommand (16 tests) - COMPLETE
  - ⏳ Phase 5: ClaudeBridge dispatcher tests (~15 tests)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ManageXR/claude-unity-bridge](https://github.com/ManageXR/claude-unity-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
