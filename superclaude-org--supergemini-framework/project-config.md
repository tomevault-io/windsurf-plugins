---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SuperGemini is a meta-programming framework that enhances Gemini CLI with structured development capabilities. It operates through **behavioral instruction injection** - modifying AI behavior via configuration files rather than code changes.

**Key Architecture Pattern**: This is NOT a traditional Python application. SuperGemini is a **configuration distribution system** that:
1. Installs behavioral instruction files (`.md`, `.toml`) to `~/.gemini/`
2. These files modify Gemini CLI behavior through instruction injection
3. Python code (`setup/`, `SuperGemini/`) handles installation/management, not runtime execution

**Package Aliases**: The CLI is accessible via three entry points:
- `SuperGemini` (primary, capitalized)
- `supergemini` (lowercase alias)
- `sg` (short form)

Documentation MUST use `SuperGemini` (capitalized) for consistency. See `pyproject.toml:44-47` for authoritative source.

## Development Commands

### Installation & Setup
```bash
# Development installation (editable mode)
pip install -e .

# Install with pipx (production-like testing)
pipx install .

# Run the CLI directly
python -m SuperGemini install --help
SuperGemini --version
```

### Testing
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_mcp_prerequisites.py

# Run with coverage
pytest --cov=SuperGemini --cov=setup --cov-report=html

# Run tests matching pattern
pytest -k "test_find_node"
```

### Code Quality
```bash
# Format code with black
black SuperGemini/ setup/ tests/

# Lint with flake8
flake8 SuperGemini/ setup/ --max-line-length=88

# Type checking with mypy
mypy SuperGemini/ setup/
```

### Building & Distribution
```bash
# Build distribution packages
python -m build

# Check distribution
twine check dist/*

# Test installation from built package
pip install dist/SuperGemini-*.whl
```

## Critical Architecture Concepts

### 1. Component-Based Installation System

**Location**: `setup/components/`

SuperGemini installs **six component types** to `~/.gemini/`:

```
setup/components/
├── core.py        # Core framework files (PRINCIPLES.md, RULES.md, FLAGS.md)
├── commands.py    # /sg: slash commands (18 TOML files → ~/.gemini/commands/sg/)
├── modes.py       # Behavioral modes (brainstorming, introspection, etc.)
├── mcp.py         # MCP server configurations (context7, sequential, magic, etc.)
├── mcp_docs.py    # MCP documentation files
```

**Key Pattern**: Each component extends `setup/core/base.py:Component` abstract class:
- `get_metadata()` - component identity
- `validate_prerequisites()` - installation requirements
- `install()` - copy files to ~/.gemini
- `uninstall()` - remove component files
- `update()` - version management

### 2. Dual Directory Structure

**CRITICAL**: This project has TWO primary directories with similar names but different purposes:

**`SuperGemini/`** (Package Source):
- **Purpose**: Source files that get COPIED during installation
- Contains: Commands/, Agents/, Modes/, Core/, Config/, MCP/
- These are `.md` and `.toml` files that modify Gemini CLI behavior
- **Never executed directly** - they are configuration/instruction files

**`setup/`** (Installation Engine):
- **Purpose**: Python code that performs the installation
- Contains: components/, core/, services/, utils/
- This IS the executable code that runs during `SuperGemini install`
- Handles file copying, validation, configuration management

**Mental Model**:
- `SuperGemini/` = "payload" (what gets installed)
- `setup/` = "installer" (how it gets installed)

### 3. Command Namespace: `/sg:` vs `SuperGemini`

**Two distinct command spaces**:

**Terminal Commands** (installer management):
```bash
SuperGemini install --yes         # Runs setup/core/installer.py
SuperGemini update                # Updates installed components
SuperGemini uninstall             # Removes ~/.gemini files
```

**Gemini CLI Commands** (after installation):
```
/sg:analyze src/                  # Uses ~/.gemini/commands/sg/analyze.toml
/sg:implement "feature"           # Uses ~/.gemini/commands/sg/implement.toml
/sg:save checkpoint              # Uses ~/.gemini/commands/sg/save.toml
```

**Critical Distinction**: `/sg:` commands are NOT Python functions. They are TOML configuration files that inject prompts into Gemini CLI.

### 4. Instruction Injection Mechanism

**How commands work** (`setup/components/commands.py:377-463`):

1. Source: `SuperGemini/Commands/analyze.md` (Markdown with front matter)
2. Conversion: `_convert_md_to_toml()` converts to TOML format
3. Installation: Copied to `~/.gemini/commands/sg/analyze.toml`
4. Gemini CLI reads TOML and injects prompt when `/sg:analyze` is typed

**TOML Structure**:
```toml
prompt = """SuperGemini Framework Command: /sg:analyze

[Full command instructions here...]

When handling flags:
- --seq: Activate sequential-thinking MCP server
- --c7: Activate context7 MCP server
"""

description = "Analyze codebase architecture"
```

### 5. Agent System (Persona Mode)

**Location**: `SuperGemini/Agents/` (13 specialized agent files)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuperClaude-Org/SuperGemini_Framework](https://github.com/SuperClaude-Org/SuperGemini_Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
