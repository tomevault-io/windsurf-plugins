---
trigger: always_on
description: **Analysis Date:** 2026-02-19
---

# Coding Conventions

**Analysis Date:** 2026-02-19

## Naming Patterns

**Files:**
- Module files use `snake_case`: `activationresult.py`, `constants.py`, `mode.py`
- Platform-specific method files: `windows.py`, `macos.py`, `gnome.py`, `freedesktop.py`
- Test files follow pattern: `test_*.py` or `*_test.py`
- Special test utilities: `conftest.py` for pytest fixtures, `testmethods.py` for test helpers

**Functions:**
- Lowercase with underscores: `activate_method()`, `get_platform_supported()`, `is_env_var_truthy()`
- Private functions prefixed with underscore: `_do_assert()`, `_check_supported_platforms()`
- Helper functions at module level before class definitions

**Variables:**
- Local variables: `snake_case` (e.g., `inhibit_cookie`, `method_inhibit`, `exit_event`)
- Constants: `UPPERCASE_WITH_UNDERSCORES` (e.g., `WAKEPY_FAKE_SUCCESS_METHOD`, `FALSY_ENV_VAR_VALUES`, `XDG_SESSION_DESKTOP`)
- Class variables use standard naming: `name`, `mode_name`, `supported_platforms` (matching enum member names)

**Types:**
- Classes: `PascalCase` (e.g., `Method`, `Mode`, `ActivationResult`, `DBusAdapter`)
- Enums: `PascalCase` (e.g., `IdentifiedPlatformType`, `PlatformType`, `ModeName`, `MethodOutcome`)
- Type aliases: `PascalCase` (e.g., `MethodCls = Type["Method"]`, `MethodOutcomeValue = Literal[...]`)
- Protocol/ABC classes: `PascalCase` (e.g., `Method`, `DBusAdapter`)

## Code Style

**Formatting:**
- Tool: `ruff` (with `fix = true` in `pyproject.toml`)
- Line length: 88 characters (Black standard)
- File imports at top in groups separated by blank lines
- Max docstring length: 79 characters

**Linting:**
- Tool: `ruff` with strict rules enabled
- Rules enforced: BLE, E, ERA, F, FIX (todo/fixme), FBT, I, S, W505, W291
- Special per-file rules:
  - Tests: S101 (assert), BLE (blind-except), FBT (boolean-trap) disabled
  - All pragmatic `# noqa` comments are acceptable with reasons

**Comments:**
- Mark pragma directives for platform/version-specific code:
  - `# pragma: no-cover-if-py-gte-38` - Skip coverage on Python >= 3.8
  - `# pragma: no-cover-if-py-lt-38` - Skip coverage on Python < 3.8
  - `# pragma: no-cover-if-no-dbus` - Skip on non-Linux (no D-Bus)
- These allow 100% coverage across all Python versions/platforms without false negatives

## Import Organization

**Order:**
1. `from __future__ import annotations` (always first for Python 3.7+ compatibility)
2. Standard library imports (`sys`, `os`, `typing`, `logging`, etc.) with `import` statements
3. Third-party imports (`pytest`, `jeepney`)
4. Local imports (relative or absolute `wakepy.*` imports)
5. `if typing.TYPE_CHECKING:` block for expensive/circular imports

**Path Aliases:**
- No path aliases configured; imports use absolute paths from package root
- Absolute imports preferred: `from wakepy.core import Method, Mode`
- Relative imports in tests: `from tests.unit.test_core.testmethods import TestMethod`

**Example from `src/wakepy/core/mode.py`:**
```python
from __future__ import annotations

import logging
import threading
import typing
import warnings
from contextvars import ContextVar
from dataclasses import dataclass, field
from functools import wraps

from wakepy.core.constants import WAKEPY_FAKE_SUCCESS_METHOD, StageName
from wakepy.core.platform import CURRENT_PLATFORM, get_platform_supported

from .activationresult import (
    ActivationResult,
    MethodActivationResult,
    ProbingResults,
)
from .dbus import DBusAdapter, get_dbus_adapter
from .heartbeat import Heartbeat
from .method import Method, MethodInfo, activate_method, deactivate_method
from .prioritization import order_methods_by_priority
from .registry import get_method, get_methods_for_mode
from .utils import is_env_var_truthy

if typing.TYPE_CHECKING:
    import sys
    from contextvars import Token
    from types import TracebackType
    from typing import Callable, List, Optional, Tuple, Type, Union
    # ... more TYPE_CHECKING imports
```

## Error Handling

**Patterns:**
- Custom exceptions inherit from standard library base classes:
  - `ActivationError(RuntimeError)` - For activation failures
  - `ActivationWarning(UserWarning)` - For activation warnings
  - `NoMethodsWarning(UserWarning)` - For missing methods
  - `ThreadSafetyWarning(UserWarning)` - For thread issues
  - `ModeExit(Exception)` - For exiting mode blocks
  - `DBusCallError(RuntimeError)` - For D-Bus failures
  - `ContextAlreadyEnteredError(RuntimeError)` - For mode state issues

- Raise with descriptive messages including context:
  ```python
  raise RuntimeError(f"Could not get inhibit cookie from {self.name}")
  raise ValueError("Methods without a name may not be used to activate modes!")
  ```

- Use `pytest.raises()` context manager in tests with specific exception matching:
  ```python
  with pytest.raises(ValueError, match=re.escape("Methods without a name...")):
      activate_method(method)
  ```

- Platform-specific exceptions caught by D-Bus adapters:
  ```python
  try:
      # D-Bus call
  except Exception:
      # Log and handle gracefully
  ```

## Logging

**Framework:** `logging` standard library

**Pattern:**
- Module-level logger: `logger = logging.getLogger(__name__)`
- Use debug level for:
  - Platform detection steps: `logger.debug("Platform debug info...")`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wakepy/wakepy](https://github.com/wakepy/wakepy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
