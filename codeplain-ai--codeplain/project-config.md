---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is **codeplain**, a Python CLI tool that renders code from `***plain` specification files using the Codeplain API. The tool acts as a client that:
1. Parses `***plain` spec files (a domain-specific language for software specifications)
2. Sends specs to the Codeplain API (LLM-based code generation service)
3. Manages an iterative state-machine-driven render process with testing and refactoring cycles
4. Outputs generated code to a build folder

The codebase serves two audiences:
- **End users**: developers who write `***plain` specs and run `plain2code.py` to generate code
- **Internal devs**: maintaining the renderer itself (this codebase)

### Related Repository: plain2code_rest_api

This repository has a sibling repository **`plain2code_rest_api`** which contains the backend API service that this client communicates with. The two repositories are typically cloned as siblings:

```
parent-directory/
├── codeplain/              # This repository (client)
└── plain2code_rest_api/    # Backend API service
```

**When to work across both repositories:**
- API contract changes (request/response formats)
- Adding new endpoints or modifying existing ones
- Debugging communication issues between client and server
- End-to-end feature development that requires both client and server changes

When working on features that span both repositories, coordinate changes carefully to maintain backward compatibility or plan coordinated deployments.

## Development Setup

### Prerequisites
- Python 3.11+
- `CODEPLAIN_API_KEY` environment variable set

### Installation
```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Git Hooks
A pre-push hook runs automatically on `git push` and executes:
- Black formatting check
- isort import sorting check  
- Flake8 linting
- Mypy type checking
- Full test suite with coverage

Hook is located at `.git/hooks/pre-push`.

## Common Commands

### Running the Tool
```bash
# Basic usage - render a .plain file
python plain2code.py path/to/file.plain

# Display account status (user info, credits, API key label)
python plain2code.py --status

# Dry run (parse and validate .plain files without rendering code)
# This parses the spec files, resolves imports/requires, but doesn't generate code
python plain2code.py path/to/file.plain --dry-run

# Enable file logging (writes to codeplain.log in same dir as .plain file)
python plain2code.py path/to/file.plain --log-to-file

# Headless mode (no TUI, logs to file only)
python plain2code.py path/to/file.plain --headless

# Render specific functionality range
python plain2code.py file.plain --render-range 1,3  # Render functionalities 1-3
python plain2code.py file.plain --render-from 2     # Resume from functionality 2

# Run with examples
cd examples/example_hello_world_python
python ../../plain2code.py hello_world_python.plain
cd build && python hello_world.py
```

### Testing
```bash
# Run all tests
pytest tests/ -v

# Run specific test file
pytest tests/test_plain_modules.py -v

# Run specific test
pytest tests/test_plain_modules.py::test_get_next_frid_within_same_module -v

# Run with coverage
coverage run -m pytest tests/ -v
coverage report
coverage html  # Generates htmlcov/index.html
```

### Code Quality
```bash
# Format code
black .

# Sort imports
isort .

# Lint
flake8 .

# Type check
mypy . --check-untyped-defs

# Run all checks (same as pre-push hook)
black . --check && isort . --check-only && flake8 . && mypy . --check-untyped-defs && coverage run -m pytest tests/ -v && coverage report
```

## Architecture

### High-Level Flow
1. **Entry Point** (`plain2code.py`): CLI argument parsing, logging setup, TUI initialization
2. **Module Loading** (`plain_modules.py`, `plain_file.py`): Parse `.plain` files into `PlainModule` objects with dependency trees
3. **Rendering Orchestration** (`module_renderer.py`): Coordinates render process across modules
4. **State Machine** (`render_machine/`): Hierarchical state machine drives the render lifecycle
5. **API Communication** (`codeplain_REST_api.py`): HTTP client for Codeplain API
6. **Code Generation** (`render_machine/code_renderer.py`): Main code renderer class that orchestrates the code generation workflow using a hierarchical state machine
7. **TUI** (`tui/`): Textual-based terminal UI showing render progress
8. **CLI Output** (`cli_output/`): Non-interactive terminal output formatting for status, dry-run, and render summaries

### Key Concepts

**Plain Modules**: A `.plain` file can `import` other `.plain` files, creating a module dependency tree. The renderer processes required modules first (depth-first), then the top module.

**Functionalities (FRIDs)**: Each functional requirement in a `.plain` file has a unique ID (FRID). The renderer processes them sequentially. State is checkpointed after each FRID.

**Render State Machine** (`render_machine/states.py`): Hierarchical FSM with states like:
- `IMPLEMENTING_FRID` → `PROCESSING_UNIT_TESTS` → `REFACTORING_CODE` → `PROCESSING_CONFORMANCE_TESTS`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Codeplain-ai/codeplain](https://github.com/Codeplain-ai/codeplain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
