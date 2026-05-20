---
trigger: always_on
description: @../rfc/standards/python-coding.md
---

@../rfc/standards/python-coding.md
@../rfc/standards/python-testing.md
@../rfc/standards/architecture.md
@../rfc/standards/git-conventions.md

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

StreetRace🚗💨 is an agentic AI coding partner designed to help engineers leverage AI capabilities directly from the command line. It serves as a bridge between AI language models and project resources, enabling AI to take actions on various tasks, with the main focus being Developer Experience and Productivity, Software Delivery, and DevOps.

## Common Development Commands

### Environment Setup

- **Install dependencies**: `poetry install`
- **Run StreetRace**: `poetry run streetrace --model=$YOUR_FAVORITE_MODEL`

### Development Workflow

- **Run tests**: `poetry run pytest tests -vv --no-header --timeout=5 -q` or `make test`
- **Run single test**: `poetry run pytest tests/path/to/test_file.py::test_function_name -v`
- **Lint code**: `poetry run ruff check src tests --ignore=FIX002` or `make lint`
- **Type checking**: `poetry run mypy src` or `make typed`
- **Security scan**: `poetry run bandit -r src` or `make security`
- **Check dependencies**: `poetry run deptry src tests` or `make depcheck`
- **Find unused code**: `poetry run vulture src vulture_allow.txt` or `make unusedcode`
- **Run all checks**: `make check` (runs test, lint, typed, security, depcheck, unusedcode)

**vulture_allow.txt policy**: Entries in `vulture_allow.txt` may ONLY be added for structurally rational reasons where code is genuinely used but vulture cannot detect it (e.g., Lark transformer methods called by name, Pydantic validators, dynamically dispatched functions, dataclass fields, public API surface). NEVER add entries to suppress warnings about actually dead code - instead, delete the dead code. Each entry MUST include a comment explaining why the code appears unused to vulture but is actually reachable.

**Important**: Run `make check` frequently during development and address all reported issues immediately before committing.

### Coverage and Profiling

- **Generate coverage report**: `make coverage`
- **Profile startup**: `poetry run python scripts/profile_startup.py`
- **Compare profiles**: `poetry run python scripts/compare_profiles.py`

## Architecture Overview

StreetRace follows a modular, layered architecture with clear separation of concerns:

### Core Components

**Workflow Layer (`workflow/supervisor.py`)**

- Orchestrates interaction loop between users and AI agents
- Manages ADK sessions and conversation persistence
- Handles tool integration and event processing

**Agent Management (`agents/agent_manager.py`)**

- Discovers, validates, and creates specialized AI agents
- Supports both modern StreetRaceAgent interface and legacy function-based agents
- Manages agent lifecycle with proper dependency injection

**LLM Integration (`llm/`)**

- `model_factory.py`: Factory for creating and caching language model instances
- `llm_interface.py`: Abstraction layer for various LLM providers
- `lite_llm_client.py`: Resilient client with retry logic and cost tracking

**UI Layer (`ui/`)**

- `console_ui.py`: Terminal-based interface with rich text and interactive prompts
- `ui_bus.py`: Event-driven communication system using pub/sub pattern
- `completer.py`: Intelligent auto-completion for files and commands

**Tool System (`tools/`)**

- `tool_provider.py`: Centralizes tool discovery and MCP integration
- `fs_tool.py`: Safe file system operations within working directory
- `cli_tool.py` + `cli_safety.py`: Controlled CLI command execution with security analysis
- `agent_tools.py`: Agent discovery and management tools

### Key Patterns

- **Dependency Injection**: Used throughout for testability and modularity
- **Event-Driven Architecture**: UI components communicate via UiBus events
- **Factory Pattern**: For model and agent creation
- **Command Pattern**: For internal commands (`commands/command_executor.py`)
- **Security by Design**: CLI safety analysis prevents dangerous operations

## Code Style Guidelines

When implementing code, produce a clean final implementation that is ready for testing
and production.

### Python Standards

- NEVER use the word Legacy.
- Use type annotations for all functions
- Provide docstrings for public symbols
- Use imperative mood for the first line of docstrings.
- Use absolute imports (`from streetrace... import ...`)
- Use double quotes for strings
- Keep functions under McCabe complexity 10
- Use module-level logger: `streetrace.log.get_logger(__name__)`
- When logging, ensure deferred formatting by passing values as arguments to the logging
  method.
- Use logging.exception when logging exceptions.
- Introduce descriptive constants instead of magic values in comparisons and document
  constants using a docstring.
- Use a single `with` statement with multiple contexts instead of nested `with`
  statements.
- Keep newline at end of file.
- Always run `ruff` on the changed files.
- When raising exceptions, assign the message to a variable first.
- Create small clearly isolated and testable modules with dependency injection.
- Avoid boolean positional arguments in method definitions - use keyword-only arguments or enums instead.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [streetrace-ai/streetrace](https://github.com/streetrace-ai/streetrace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
