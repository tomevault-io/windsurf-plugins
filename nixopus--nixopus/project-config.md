---
trigger: always_on
description: Python CLI Development Rules for Nixopus CLI
---


# Nixopus CLI Development Guidelines

You are a senior Python engineer building the Nixopus CLI — a production-grade command-line tool using Typer, Rich, and Pydantic. Your focus is on writing clean, maintainable, and user-friendly CLI commands following established patterns.

## Core Principles

### DRY (Don't Repeat Yourself) — Highest Priority
- **Before writing new code**, search the codebase for existing implementations
- Check `app/utils/` for shared utilities (`logger`, `config`, `output_formatter`, `timeout`)
- Check `app/utils/protocols.py` for protocol definitions
- Reuse existing message patterns from `messages.py` files
- Extract common validation logic to shared validators

### Single Responsibility Principle (SRP)
- **Commands**: CLI interface and argument parsing only
- **Run/Logic files**: Business logic and orchestration
- **Messages**: User-facing strings (separated from logic)
- **Types**: Data classes and type definitions
- **Utils**: Reusable utility functions
- Each file should have one primary purpose

### Code Readability
```python
# ✅ Good: Early returns, flat structure
def clone_repository(repo: str, path: str, logger: LoggerProtocol) -> tuple[bool, Optional[str]]:
    if not repo:
        return False, "Repository URL is required"
    
    if not validate_repo_url(repo):
        return False, "Invalid repository URL"
    
    try:
        result = git_clone(repo, path)
        return True, None
    except Exception as e:
        return False, str(e)

# ❌ Bad: Nested conditions
def clone_repository(repo: str, path: str, logger: LoggerProtocol) -> tuple[bool, Optional[str]]:
    if repo:
        if validate_repo_url(repo):
            try:
                result = git_clone(repo, path)
                return True, None
            except Exception as e:
                return False, str(e)
        else:
            return False, "Invalid repository URL"
    else:
        return False, "Repository URL is required"
```

## Architecture

### Directory Structure
```
cli/
├── app/
│   ├── __init__.py
│   ├── main.py                 # Entry point, Typer app registration
│   ├── commands/               # Command modules
│   │   └── [command]/
│   │       ├── __init__.py
│   │       ├── command.py      # Typer command definitions
│   │       ├── messages.py     # User-facing strings
│   │       ├── types.py        # Dataclasses & types
│   │       └── [logic].py      # Business logic
│   └── utils/                  # Shared utilities
│       ├── __init__.py
│       ├── config.py           # Configuration loading
│       ├── logger.py           # Logging utilities
│       ├── message.py          # Global messages
│       ├── output_formatter.py # Output formatting
│       ├── protocols.py        # Protocol definitions
│       └── timeout.py          # Timeout utilities
├── pyproject.toml              # Poetry configuration
└── tests/                      # Test files
```

### Creating a New Command

1. Create the directory structure:
```
app/commands/[command]/
├── __init__.py
├── command.py      # or [command].py
├── messages.py
└── types.py        # if needed
```

2. Register in `app/main.py`:
```python
from app.commands.[command].command import [command]_app

app.add_typer([command]_app, name="[command]")
```

## Typer Command Patterns

### Command File Structure
```python
from typing import Optional
import typer
from app.utils.logger import create_logger, log_error, log_success
from app.utils.timeout import timeout_wrapper
from .messages import operation_failed, operation_success
from .types import CommandParams

command_app = typer.Typer(help="Command description", invoke_without_command=True)


@command_app.callback()
def command_callback(
    ctx: typer.Context,
    verbose: bool = typer.Option(False, "--verbose", "-v", help="Show more details"),
    timeout: int = typer.Option(300, "--timeout", "-t", help="Timeout in seconds"),
    dry_run: bool = typer.Option(False, "--dry-run", "-d", help="Preview changes without executing"),
    force: bool = typer.Option(False, "--force", "-f", help="Force operation"),
):
    """Main command description"""
    if ctx.invoked_subcommand is None:
        logger = create_logger(verbose=verbose)
        params = CommandParams(
            logger=logger,
            verbose=verbose,
            timeout=timeout,
            dry_run=dry_run,
            force=force,
        )
        run_command(params)


@command_app.command(name="subcommand")
def subcommand(
    arg: str = typer.Argument(..., help="Required argument"),
    option: str = typer.Option(None, "--option", "-o", help="Optional argument"),
    verbose: bool = typer.Option(False, "--verbose", "-v", help="Verbose output"),
):
    """Subcommand description"""
    logger = create_logger(verbose=verbose)
    try:
        with timeout_wrapper(timeout):
            result = execute_subcommand(arg, option)
        log_success(operation_success, verbose=verbose)
    except TimeoutError as e:
        log_error(str(e), verbose=verbose)
        raise typer.Exit(1)
    except Exception as e:
        log_error(str(e), verbose=verbose)
        raise typer.Exit(1)
```

### Standard CLI Options
```python
# Always include these common options where applicable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nixopus/nixopus](https://github.com/nixopus/nixopus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
