---
trigger: always_on
description: The Tinker CLI is a command-line interface for the Tinker SDK, designed with a focus on fast startup times, modular architecture, and user-friendly output formats. The CLI uses Click framework with custom lazy loading to maintain performance.
---

# Tinker CLI Design Documentation

## Overview

The Tinker CLI is a command-line interface for the Tinker SDK, designed with a focus on fast startup times, modular architecture, and user-friendly output formats. The CLI uses Click framework with custom lazy loading to maintain performance.

## Key Design Decisions

### 1. Lazy Import Strategy with Click

**Decision**: Use Click framework with a custom `LazyGroup` class for lazy loading. Only Click is imported at the module level.

**Rationale**: This ensures that `tinker --help` is lightning fast (<50ms startup time). Users shouldn't have to wait for heavy imports when they just want to see available commands.

**Implementation**:
- Main `__init__.py` only imports `click` and `lazy_group`
- Command modules are loaded only when invoked via `LazyGroup`
- Output formatting imports `rich` only when table output is needed
- JSON module imported only when JSON output is requested
- Version information loaded from `_version.py` only when `tinker version` is used

### 2. Click Framework with LazyGroup

**Decision**: Migrated from argparse to Click, implementing a custom `LazyGroup` class that extends Click's Group to support lazy loading.

**Rationale**:
- Click provides cleaner command structure with decorators
- Better subcommand isolation - each command file is self-contained
- Automatic help generation with better formatting
- Built-in type conversion and validation
- LazyGroup enables fast startup by deferring imports

**LazyGroup Implementation**:
```python
class LazyGroup(click.Group):
    def __init__(self, *args, lazy_subcommands=None, **kwargs):
        # Map of command name to "module.path:command_name"
        self.lazy_subcommands = lazy_subcommands or {}

    def get_command(self, ctx, cmd_name):
        if cmd_name in self.lazy_subcommands:
            # Import only when command is actually invoked
            import_path = self.lazy_subcommands[cmd_name]
            module_name, attr_name = import_path.rsplit(":", 1)
            mod = importlib.import_module(module_name)
            return getattr(mod, attr_name)
```

### 3. Hierarchical Command Structure

**Decision**: Commands are organized hierarchically with main commands and subcommands (e.g., `tinker run list`, `tinker checkpoint info`), plus standalone commands like `tinker version`.

**Rationale**:
- Provides a consistent, predictable interface
- Groups related functionality together
- Makes the CLI extensible for future commands
- Follows common CLI patterns (like `git`, `docker`, etc.)

**Examples**:
- `tinker version` - Show CLI and SDK version
- `tinker run list` - List all training runs
- `tinker run info <run-id>` - Show details of a specific run
- `tinker checkpoint list` - List all checkpoints
- `tinker checkpoint info <checkpoint-id>` - Show checkpoint details
- `tinker checkpoint push-hf <checkpoint-path>` - Upload a checkpoint to Hugging Face Hub

### 4. Output System with Inheritance

**Decision**: Use an abstract base class (`OutputBase`) that all command outputs inherit from. Each command defines its own output class.

**Rationale**:
- Enforces consistent interface across all commands
- Encapsulates output logic with the command that generates it
- Makes it easy to support multiple output formats (table, JSON)
- Keeps related code together in the same module

**Implementation**:
- `OutputBase` in `output.py` defines the contract
- Each command module contains its own output classes (e.g., `RunListOutput`, `RunInfoOutput`)
- Base class handles format selection and rendering

### 5. Self-Contained Command Modules

**Decision**: Each command is a self-contained Click command/group in its own file with a `cli` entry point.

**Rationale**:
- Modular architecture - commands can be developed independently
- Clear separation of concerns
- Easy to add new commands without modifying core files
- Consistent pattern across all commands

**Command Structure**:
```python
# Each command file follows this pattern:
@click.group()  # or @click.command() for simple commands
def cli():
    """Command description."""
    pass

@cli.command()  # For subcommands
def list():
    """Subcommand implementation."""
    pass
```

### 6. Centralized Client Management

**Decision**: All SDK client creation and error handling is centralized in `client.py`.

**Rationale**:
- Single place to handle authentication and connection errors
- Consistent error messages across all commands
- Reusable error handling decorator
- Clean separation of concerns

### 7. Rich Tables for Human-Readable Output

**Decision**: Use the `rich` library for table formatting, kept as an optional dependency.

**Rationale**:
- Provides beautiful, formatted tables with colors and borders
- Handles column width adjustment automatically
- Supports both dark and light terminal themes
- Optional dependency keeps the core package lightweight

### 8. Unix-Style Default Output

**Decision**: Default output is human-readable tables, with `--format json` flag for machine-readable output.

**Rationale**:
- Follows Unix philosophy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thinking-machines-lab/tinker](https://github.com/thinking-machines-lab/tinker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
