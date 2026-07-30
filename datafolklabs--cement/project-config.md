---
trigger: always_on
description: **Analysis Date:** 2026-04-24
---

# Coding Conventions

**Analysis Date:** 2026-04-24

## Naming Patterns

**Files:**
- Module files: lowercase with underscores (`foundation.py`, `ext_smtp.py`)
- Classes: PascalCase (e.g., `FrameworkError`, `SMTPMailHandler`, `MetaMixin`)
- Test files: `test_<module>.py` pattern (e.g., `test_foundation.py`, `test_ext_smtp.py`)

**Functions:**
- Public methods: lowercase_with_underscores (`parse_file()`, `_setup_arg_handler()`)
- Private/internal methods: leading underscore (`_get_params()`, `_parse_file()`)
- Test functions: `test_<feature>()` pattern (e.g., `test_smtp_send()`, `test_basic()`)

**Variables:**
- Constants: UPPERCASE_WITH_UNDERSCORES (environment variables, signal lists)
- Instance variables: lowercase_with_underscores
- Module-level logger: `LOG = minimal_logger(__name__)`

**Types:**
- Use PascalCase for class names
- Use **PEP 585 builtin generics** (Python 3.10+): `dict[str, Any]`, `list[str]`, `tuple[int, str]`, `type[Handler]`
- Use **PEP 604 union syntax** (Python 3.10+): `str | None`, `int | str`, `dict[str, Any] | None`
- Phase 03 D-06 enabled ruff `UP` family enforcement; UP006/UP007/UP045 mechanically rewrite legacy syntax to the modern forms on every `make comply-ruff-fix`
- Private Meta attribute annotations: `_meta: MetaClassName  # type: ignore` (see `cement/ext/ext_smtp.py` line 72)

## Code Style

**Formatting:**
- Tool: `ruff` (v0.3.2+)
- Line length: 100 characters
- Indentation: 4 spaces
- Python target: 3.9+

**Ruff Configuration** (`pyproject.toml` [tool.ruff]):
```
target-version = "py310"
line-length = 100
indent-width = 4
preview = true
extend-select = ["E", "F", "W"]  # PEP8, Pyflakes, warnings
fixable = ["ALL"]
```

**Linting:**
- Tool: `ruff check cement/ tests/`
- All code must pass without errors before commit
- Run via `make comply-ruff`

## Import Organization

**Order** (strictly enforced):
1. Standard library imports (`os`, `sys`, `signal`, `platform`, `importlib`)
2. Third-party library imports (if any)
3. Relative imports from cement (`from ..core`, `from ..utils`)
4. TYPE_CHECKING block (deferred type imports to avoid circular dependencies)

**Example** from `cement/core/foundation.py`:
```python
from __future__ import annotations
import os
import platform
import signal
import sys
from importlib import reload as reload_module
from time import sleep
from typing import (IO, Any, Callable, Dict, List, Optional, TextIO, Tuple,
                    Type, Union, TYPE_CHECKING)
from ..core import (arg, cache, config, controller, exc, extension, log, mail,
                    meta, output, plugin, template)
from ..core.deprecations import deprecate
from ..utils.misc import is_true, minimal_logger

if TYPE_CHECKING:
    from types import FrameType, ModuleType, TracebackType  # pragma: nocover
```

**Path Aliases:**
- No centralized alias configuration; use relative imports exclusively (`..core.`, `..utils.`)

## Type Annotations

**Strictness:** Full type annotation compliance required
- mypy config in `pyproject.toml` enforces strict mode:
  - `disallow_untyped_calls = true`
  - `disallow_untyped_defs = true`
  - `disallow_incomplete_defs = true`
  - `warn_return_any = true`

**Patterns:**
- Always annotate function parameters and return types: `def send(self, msg: str, **kw: Any) -> dict[str, Any]:`
- Use **PEP 585 builtin generics** (Python 3.10+): `dict[str, Any]`, `list[str]`, `tuple[int, str]`, `type[Handler]`
- Use **PEP 604 union syntax** (Python 3.10+): `str | None`, `int | str`, `dict[str, Any] | None`
- Use `TYPE_CHECKING` block for deferred imports to prevent circular dependencies
- Private Meta attributes require `# type: ignore` due to metaclass pattern (framework constraint)
- Phase 03 D-06 enabled ruff `UP` family enforcement; UP006/UP007/UP045 mechanically rewrite legacy syntax to the modern forms on every `make comply-ruff-fix`. Phase 03 plan 03 landed the bulk migration.

**Example** from `cement/ext/ext_smtp.py` (post Phase 03):
```python
def _get_params(self, **kw: Any) -> dict[str, Any]:
    params = dict()
    # ...
    return params
```

## Error Handling

**Framework Exceptions** (from `cement/core/exc.py`):
- `FrameworkError` - General framework (non-application) errors with message passing
- `InterfaceError` - Interface-related errors
- `CaughtSignal(signum, frame)` - Raised when signal is caught

**Pattern:**
- Raise framework exceptions for framework-level errors
- Custom applications should extend `FrameworkError` for app-specific exceptions
- Always include a message: `raise FrameworkError("descriptive message")`

**Exception Testing** (from `tests/core/test_exc.py`):
```python
with raises(FrameworkError, match=".*framework exception.*"):
    raise FrameworkError("test framework exception message")
```

## Logging

**Framework:** `cement.utils.misc.minimal_logger()`
- Module-level logger created as: `LOG = minimal_logger(__name__)`
- No logging.config setup needed; minimal logger is intentionally simple for framework

**Patterns:**
- Use `LOG.debug()` for framework internals: `LOG.debug(f'hook {hook_spec[0]} not defined')`
- Framework runs silently by default; debug output only when app.debug is True
- No INFO/WARNING/ERROR logging in framework core (by design)

**Example** from `cement/core/foundation.py`:
```python
LOG = minimal_logger(__name__)
# ...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datafolklabs/cement](https://github.com/datafolklabs/cement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
