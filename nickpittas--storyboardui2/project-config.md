---
trigger: always_on
description: This document provides guidelines for AI agents working on the Storyboard Maker application.
---

# AGENTS.md - Storyboard Maker Development Guide

This document provides guidelines for AI agents working on the Storyboard Maker application.

## Project Overview

A Python/PyQt6 desktop application that interfaces with ComfyUI for AI image generation. Enables filmmakers to create storyboards with camera angle control, LoRA integration, and reference image support.

**Tech Stack**: Python 3.10+, PyQt6 6.6+, requests 2.31+, Pillow 10.0+, jsonschema 4.19+, ReportLab 4.0+

## Build Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run application (from project root)
python -m storyboard_app.main

# Development mode with hot reload (if implemented)
python -m storyboard_app.main --dev

# Linting
flake8 storyboard_app/ --max-line-length=100
black storyboard_app/ --line-length 100
isort storyboard_app/ --profile black

# Type checking
mypy storyboard_app/

# Tests
pytest tests/                    # Run all tests
pytest tests/ -v               # Verbose output
pytest tests/ -k "test_name"   # Run single test by name
pytest tests/ --tb=short       # Short traceback
pytest tests/ --cov=storyboard_app  # With coverage
```

## Code Style Guidelines

### Imports

```python
# Standard library first, then third-party, then local
import json
import pathlib
from pathlib import Path
from typing import Any, Optional

import jsonschema
import requests
from PyQt6.QtCore import QObject, pyqtSignal
from PyQt6.QtWidgets import QMainWindow, QWidget

from .config import Config
from .models.template import Template
```

**Rules**:
- Use absolute imports for package consistency
- Sort imports within each group alphabetically
- Avoid wildcard imports (`from x import *`)
- Use `from typing import ...` for type hints, not typing module prefix

### Formatting

- **Line length**: 100 characters maximum
- **Indentation**: 4 spaces (no tabs)
- **Blank lines**: 2 between class definitions, 1 between function definitions
- **Quotes**: Double quotes for strings, single quotes only when containing double quotes
- **Trailing commas**: Required for multi-line lists/dicts

### Type Hints

```python
# Always use type hints for function signatures
def load_template(self, path: Path) -> Optional[Template]:
    ...

# Use Literal for enum-like strings
from typing import Literal
EngineType = Literal["qwen", "z_image", "flux", "sd"]

# Use TypedDict for config structures
from typing import TypedDict
class ComfyUIConfig(TypedDict):
    server_url: str
    timeout: int
    max_retries: int

# Avoid Any - be specific about types
# BAD: def process(data: Any) -> Any:
# GOOD: def process(data: dict[str, Any]) -> list[str]:
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Modules | `snake_case` | `template_loader.py` |
| Classes | `PascalCase` | `StoryboardPanel` |
| Functions | `snake_case` | `load_workflow()` |
| Variables | `snake_case` | `comfyui_client` |
| Constants | `SCREAMING_SNAKE_CASE` | `DEFAULT_TIMEOUT` |
| Private methods | `_snake_case` | `_validate_schema()` |
| Qt widgets | Suffix with widget type | `MainWindow`, `AngleSelector` |

### Error Handling

```python
# Use custom exceptions hierarchy
class StoryboardError(Exception):
    """Base exception for application errors."""
    pass

class TemplateError(StoryboardError):
    """Template loading or validation errors."""
    pass

class ComfyUIError(StoryboardError):
    """ComfyUI API communication errors."""
    pass

# Handle errors with specific exceptions, not bare except
try:
    template = self._loader.load(path)
except TemplateError as e:
    self._logger.error(f"Failed to load template: {e}")
    return None
except OSError as e:
    raise ComfyUIError(f"File system error: {e}") from e

# Always use context managers for resources
with self._client.session() as session:
    response = session.post(url, json=payload)
```

### PyQt6 Patterns

```python
# Use modern signal syntax (PyQt6)
from PyQt6.QtCore import QObject, pyqtSignal, pyqtSlot

class ImageGenerator(QObject):
    progress = pyqtSignal(int)  # Public signal
    finished = pyqtSignal(list[Path])
    
    @pyqtSlot(str)
    def generate(self, prompt: str) -> None:
        ...

# Widget creation - use classes, not lambdas
# BAD: button.clicked.connect(lambda: self.on_click())
# GOOD: button.clicked.connect(self.on_click)

# Use QApplication.instance() for global access
app = QApplication.instance()
```

### File Organization

```
storyboard_app/
├── main.py              # Entry point
├── app.py               # Application class
├── config.py            # Configuration management
├── core/
│   ├── __init__.py
│   ├── template_loader.py
│   ├── workflow_builder.py
│   ├── comfyui_client.py
│   ├── angle_library.py
│   ├── export_manager.py
│   ├── prompt_builder.py
│   └── session_manager.py
├── models/
│   ├── __init__.py
│   ├── template.py
│   ├── parameter.py
│   ├── lora.py
│   └── image_input.py
├── ui/
│   ├── __init__.py
│   ├── main_window.py
│   ├── panels/
│   │   ├── __init__.py
│   │   ├── template_selector.py
│   │   ├── settings_panel.py
│   │   └── ...
│   └── widgets/
│       ├── __init__.py
│       ├── panel_slot.py
│       └── value_widgets.py
├── templates/           # Built-in templates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NickPittas/StoryboardUI2](https://github.com/NickPittas/StoryboardUI2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
