---
trigger: always_on
description: Provides undo/restore functionality for AI changes.
---

# AGENTS.md - Aye Chat Project Guidelines

This file provides context for AI coding assistants working on the Aye Chat codebase.

## Project Overview

**Aye Chat** is an AI-powered terminal workspace that brings AI directly into command-line workflows. It allows developers to edit files, run commands, and chat with their codebase without leaving the terminal.

### Core Philosophy

- **Optimistic workflow**: Files are written directly (LLM assumed correct), with instant `restore` to undo
- **Zero config**: Auto-detects project files, respects `.gitignore` and `.ayeignore`
- **Real shell**: Execute any command without leaving chat
- **Local-first**: All backups stored locally in `.aye/` directory

## Architecture

```
src/aye/
├── __main__.py          # CLI entry point (Typer app)
├── controller/          # Business logic, command handling
│   ├── repl.py          # Main chat REPL loop
│   ├── commands.py      # Command implementations
│   ├── command_handlers.py  # Individual command handlers
│   ├── llm_invoker.py   # LLM API invocation
│   ├── llm_handler.py   # LLM response processing
│   └── plugin_manager.py    # Plugin discovery and management
├── model/               # Data models, business logic
│   ├── api.py           # HTTP API client
│   ├── auth.py          # Token management (~/.ayecfg)
│   ├── config.py        # Constants, system prompt, models
│   ├── snapshot/        # File versioning (backup/restore)
│   ├── source_collector.py  # File collection with ignore patterns
│   ├── file_processor.py    # Path normalization, filtering
│   └── index_manager.py     # RAG vector database
├── presenter/           # UI output (Rich-based)
│   ├── repl_ui.py       # Chat UI components
│   ├── cli_ui.py        # CLI output formatting
│   ├── streaming_ui.py  # Streaming response display
│   └── diff_presenter.py    # Diff visualization
└── plugins/             # Plugin implementations
    ├── plugin_base.py   # Abstract base class
    ├── at_file_completer.py  # @file reference completion
    ├── completer.py     # Command/path completion
    └── shell_executor.py    # Shell command execution
```

## Coding Conventions

### Python Style

- **Python 3.10+** - Use modern syntax (type hints, `|` union, walrus operator where clear)
- **Type hints** - Required for function signatures, optional for locals
- **Docstrings** - Google style, required for public functions
- **Line length** - 100 characters soft limit
- **Imports** - Standard library, third-party, then local; alphabetized within groups

```python
# Good
from pathlib import Path
from typing import Optional, Dict, Any, List

import httpx
from rich import print as rprint

from aye.model.auth import get_user_config
from aye.presenter.repl_ui import print_error


def process_files(
    files: List[Dict[str, str]],
    root: Path,
    *,
    verbose: bool = False,
) -> Optional[str]:
    """Process files and return batch ID.
    
    Args:
        files: List of file dicts with 'file_name' and 'file_content'
        root: Project root path
        verbose: Enable verbose output
        
    Returns:
        Batch ID if successful, None otherwise
    """
```

### Error Handling

- Use specific exceptions, not bare `except:`
- Log errors with context using Rich formatting
- Graceful degradation - don't crash on non-critical errors

```python
# Good
try:
    content = file_path.read_text(encoding='utf-8')
except UnicodeDecodeError:
    # Skip binary files gracefully
    if verbose:
        rprint(f"[yellow]Skipping binary file: {file_path}[/]")
    return None
except PermissionError as e:
    rprint(f"[red]Permission denied:[/] {file_path}")
    raise
```

### Path Handling

- Always use `pathlib.Path`, never string manipulation for paths
- Use `.as_posix()` for cross-platform path strings in output
- Resolve paths against project root, not CWD

```python
# Good
from pathlib import Path

def resolve_file(file_name: str, root: Path) -> Path:
    p = Path(file_name)
    if p.is_absolute():
        return p
    return (root / p).resolve()
```

### Configuration

- User config stored in `~/.ayecfg` (flat INI-style file)
- Use `get_user_config()` / `set_user_config()` from `aye.model.auth`
- Environment variables override file config (prefix: `AYE_`)

```python
from aye.model.auth import get_user_config, set_user_config

# Reading config with default
verbose = get_user_config("verbose", "off").lower() == "on"

# Writing config
set_user_config("selected_model", "gpt-4")
```

## Testing

### Test Organization

```
tests/
├── test_*.py            # Unit tests (pytest)
├── ua/                  # User acceptance test specs (markdown)
└── e2e/                 # End-to-end tests
```

### Test Patterns

- Use `pytest` with fixtures
- Mock external dependencies (API calls, file system where needed)
- Use `tmp_path` fixture for file system tests
- Test both success and error paths

```python
import pytest
from pathlib import Path
from unittest.mock import patch, MagicMock

from aye.model.file_processor import make_paths_relative


class TestMakePathsRelative:
    def test_absolute_path_under_root(self, tmp_path):
        """Test converting absolute paths under root to relative."""
        root = tmp_path
        files = [{"file_name": str(root / "src" / "main.py")}]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acrotron/aye-chat](https://github.com/acrotron/aye-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
