---
trigger: always_on
description: Python OpenTelemetry integration for Cursor IDE hooks. Captures agent activity and exports traces to OTEL-compliant receivers. Uses LangSmith/GenAI semantic conventions for observability platforms.
---

# AGENTS.md - Cursor OTEL Hook

## Project Overview

Python OpenTelemetry integration for Cursor IDE hooks. Captures agent activity and exports traces to OTEL-compliant receivers. Uses LangSmith/GenAI semantic conventions for observability platforms.

## Quick Reference

```bash
# Setup
./setup.sh                      # Unix/macOS setup
python -m venv venv && source venv/bin/activate
pip install -e ".[dev]"         # Install with dev dependencies

# Development
black src/                      # Format code
mypy src/                       # Type check
pytest tests/                   # Run tests
pytest tests/test_foo.py        # Single test file
pytest tests/test_foo.py::test_name  # Single test function

# Manual testing
echo '{"hook_event_name":"test"}' | python -m cursor_otel_hook --debug
```

## Project Structure

```
src/cursor_otel_hook/
├── __init__.py          # Package init, version
├── __main__.py          # Entry point for -m invocation
├── config.py            # OTELConfig dataclass, config loading
├── hook_receiver.py     # Main CursorHookProcessor, CLI
├── privacy.py           # Data masking utilities
├── batching_processor.py # Generation-based span batching
├── context_manager.py   # Cross-process span relationships
└── json_exporter.py     # Custom OTLP/JSON exporter
```

## Code Style Guidelines

### Python Version
- Target Python 3.8+ (oldest supported version)
- Use features available in 3.8 (walrus operator OK, but no 3.10+ match statements)

### Formatting (Black)
- **Line length**: 100 characters
- **Quotes**: Double quotes for strings (Black default)
- Run `black src/` before committing

### Type Hints (Required)
```python
from typing import Optional, Dict, Any, Sequence, List

def process_hook(self, hook_data: Dict[str, Any]) -> Dict[str, Any]:
    """Process a hook event and create OTEL spans."""
    ...

def mask_email(email: str) -> str:
    """Mask email while preserving domain."""
    ...
```

### Imports Order
1. Standard library (`import json`, `from typing import ...`)
2. Third-party (`from opentelemetry import trace`)
3. Local imports (`from .config import OTELConfig`)

```python
import json
import logging
import sys
from pathlib import Path
from typing import Any, Dict, Optional

from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider

from .config import OTELConfig
from .privacy import mask_sensitive_data
```

### Docstrings
Use triple-quoted docstrings for all public functions and classes:

```python
def _encode_span(self, span: ReadableSpan) -> Dict[str, Any]:
    """
    Encode a single span to OTLP JSON format.
    
    Args:
        span: The OpenTelemetry span to encode
        
    Returns:
        Dictionary in OTLP JSON format
    """
```

### Dataclasses
Use `@dataclass` for configuration and data containers:

```python
from dataclasses import dataclass

@dataclass
class OTELConfig:
    """OpenTelemetry configuration"""
    endpoint: str
    service_name: str
    insecure: bool = False
    headers: Optional[dict] = None
```

### Logging
- Use module-level logger: `logger = logging.getLogger(__name__)`
- Log levels: DEBUG for verbose, INFO for operations, WARNING for recoverable issues, ERROR for failures

```python
logger = logging.getLogger(__name__)

logger.debug(f"Processing span: {span.name}")
logger.info(f"Exported {len(spans)} spans")
logger.warning(f"Config file not found, using defaults")
logger.error(f"Failed to export: {e}", exc_info=True)
```

### Error Handling
- Catch specific exceptions when possible
- Log errors with context before re-raising
- Use `exc_info=True` for stack traces in error logs

```python
try:
    config = OTELConfig.from_file(config_path)
except FileNotFoundError:
    logger.warning(f"Config not found: {config_path}, using env vars")
    config = OTELConfig.from_env()
except Exception as e:
    logger.error(f"Error loading config: {e}", exc_info=True)
    raise
```

### Naming Conventions
- **Classes**: PascalCase (`CursorHookProcessor`, `OTELConfig`)
- **Functions/Methods**: snake_case (`process_hook`, `_encode_span`)
- **Private methods**: Leading underscore (`_add_common_attributes`)
- **Constants**: SCREAMING_SNAKE_CASE at module level only
- **Variables**: snake_case (`span_data`, `hook_event`)

### Platform Compatibility
Handle Windows vs Unix differences explicitly:

```python
if sys.platform == 'win32':
    import msvcrt
    def lock_file(file_handle, exclusive=True):
        msvcrt.locking(...)
else:
    import fcntl
    def lock_file(file_handle, exclusive=True):
        fcntl.flock(...)
```

## Type Checking (mypy)

Configuration in `pyproject.toml`:
```toml
[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

**Rules**:
- All function parameters and return types must be annotated
- Use `Optional[T]` for nullable types, not `T | None` (3.10+)
- Use `Dict`, `List`, `Tuple` from typing, not built-in generics (3.8 compat)

## Testing

Tests go in `tests/` directory. Use pytest:

```python
# tests/test_config.py
import pytest
from cursor_otel_hook.config import OTELConfig


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LangGuard-AI/cursor-otel-hook](https://github.com/LangGuard-AI/cursor-otel-hook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
