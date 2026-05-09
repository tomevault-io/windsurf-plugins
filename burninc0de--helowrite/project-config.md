---
trigger: always_on
description: HeloWrite is a fast, terminal-based text editor built with Python and Textual. It provides a keyboard-driven editing experience with a clean, distraction-free interface featuring persistent themes, syntax highlighting, and integrated git functionality.
---

# HeloWrite - Agent Instructions

## Project Overview

HeloWrite is a fast, terminal-based text editor built with Python and Textual. It provides a keyboard-driven editing experience with a clean, distraction-free interface featuring persistent themes, syntax highlighting, and integrated git functionality.

### Technology Stack
- **Language**: Python 3.8+
- **UI Framework**: Textual (>=0.80.0) for building terminal user interfaces
- **Terminal Styling**: Rich (>=14.0.0) for beautiful terminal output and styling
- **File Watching**: Watchdog (>=3.0.0) for hot reload in development
- **Syntax Highlighting**: Pygments (>=2.0.0) for code syntax highlighting
- **ASCII Art**: Pyfiglet (>=1.0.0) for generating application title
- **Build System**: setuptools with pyproject.toml configuration
- **Packaging**: Standard Python packaging with entry points
- **Virtual Environment**: Managed via venv (project includes helowrite_env directory)

## Build & Development Commands

### Development
```bash
# Activate virtual environment first
source helowrite_env/bin/activate

# Then run commands
python dev.py src/app.py [filename.txt]

# Run without hot reload
python src/app.py [filename.txt]

# Alternative: use the wrapper script which handles venv activation
./run.sh python dev.py src/app.py [filename.txt]
```

### Running the TUI (⚠️ CRITICAL WARNING ⚠️)

**NEVER run the TUI directly without a timeout.** It blocks on stdin and will hang forever, killing the agent process.

⚠️ **Agent Containment Procedures**

NEVER run the TUI directly. It is a one-way trip for automated processes. The TUI consumes stdin and will not release you. You will hang until the heat death of the server.

If you must peek into the void, use a tether:

```bash
# Safety tether (3s timeout)
timeout 3 python src/app.py test.txt || echo "TUI launched (or timed out)"
```

### Testing
```bash
# Install dev dependencies
pip install -e .[dev]

# Run all tests
pytest

# Run type checking
mypy

# Run linting
ruff check src/
ruff format --check src/
```

## Code Style Guidelines

### Python Configuration
- **Type Checking**: mypy with strict=false, ignore_missing_imports=true, warn_return_any=false, warn_unused_configs=true
- **Linting**: ruff with select rules: E, W, F, I, B, C4, UP; ignore: E501, B008, C901
- **Formatting**: ruff (line-length=88, target-version=py38, quote-style=double, indent-style=space)

### Import Organization
```python
# 1. Standard library imports
import os
from pathlib import Path

# 2. Third-party imports (alphabetical)
from rich.console import Console
from textual.app import App

# 3. Local imports (grouped by directory)
from config import Config
from utils import detect_language
```

### Class and Function Structure
```python
class HeloWrite(App):
    """A simple text editor TUI application."""

    def __init__(self, file_path: Optional[str] = None):
        super().__init__()
        self.file_path = file_path
        # Initialize reactive state

    def action_save(self) -> None:
        """Save the current file."""
        # Implementation

    def compose(self) -> ComposeResult:
        """Compose the UI layout."""
        yield Header()
        yield Footer()
```

### Naming Conventions

#### Files and Directories
- **Modules**: `snake_case.py` (e.g., `config.py`, `widgets.py`)
- **Packages**: `snake_case/` (e.g., `src/`, `tests/`)
- **Test Files**: `test_*.py` (e.g., `test_config.py`)
- **Directories**: `lowercase/` (e.g., `src/`, `css/`)

#### Code Elements
- **Classes**: `PascalCase` (e.g., `HeloWrite`, `StatusBar`, `Config`)
- **Functions/Methods**: `snake_case` (e.g., `detect_language`, `action_save`, `update_status`)
- **Variables**: `snake_case` (e.g., `file_path`, `is_dirty`, `word_count`)
- **Constants**: `UPPER_CASE` (e.g., `HELP_TEXT`, `DEFAULT_WIDTH`)
- **Private Attributes**: Leading underscore (e.g., `_original_text`, `_word_count_timer`)

### Reactive Patterns

#### Textual Bindings
```python
BINDINGS = [
    Binding("ctrl+s", "save", "Save"),
    Binding("ctrl+q", "quit", "Quit"),
]

def action_save(self) -> None:
    """Handle save action."""
    # Implementation
```

#### Reactive Updates
```python
def watch_file_path(self, old_path: Optional[str], new_path: Optional[str]) -> None:
    """Watch for file path changes."""
    if new_path:
        self.load_file(new_path)
```

### Error Handling

#### Try/Except Patterns
```python
def load_file(self, path: str) -> bool:
    """Load file content, return success status."""
    try:
        with open(path, 'r', encoding='utf-8') as f:
            self.content = f.read()
        return True
    except (FileNotFoundError, PermissionError) as e:
        self.notify(f"Error loading file: {e}", severity="error")
        return False
```

#### Guard Clauses
```python
def action_save(self) -> None:
    """Save current file if path exists."""
    if not self.file_path:
        self.action_save_as()
        return
    # Continue with save logic
```

### Documentation

#### Docstrings
```python
class Config:
    """Manages application configuration with persistence."""


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [burninc0de/helowrite](https://github.com/burninc0de/helowrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
