---
trigger: always_on
description: This document provides guidelines for AI agents working on the dd-license-attribution codebase.
---

# AI Agent Guidelines for dd-license-attribution

This document provides guidelines for AI agents working on the dd-license-attribution codebase.

## 🚨 Critical Requirements

All code changes must comply with these non-negotiable requirements:

- **100% typing coverage** validated by MyPy
- **95%+ unit test coverage** for all business logic
- **OS operations through adaptors only** (no direct imports)
- **Formatted with isort and black**
- **CHANGELOG.md updates** for user-facing changes
- **SPDX-License-Identifier header** in all source files

## 📄 File Headers and Licensing

**⚠️ Always use the current calendar year** in the copyright line (e.g., 2026 for files created in 2026).

Every Python source file (`.py`) and test file must include this header at the very top:

```python
# SPDX-License-Identifier: Apache-2.0
#
# Unless explicitly stated otherwise all files in this repository are licensed under the Apache License Version 2.0.
#
# This product includes software developed at Datadog (https://www.datadoghq.com/).
# Copyright 2026-present Datadog, Inc.
```

**REQUIRED in**: all `src/` files, all `tests/` files, all Python scripts.

**NOT REQUIRED in**: empty/near-empty `__init__.py` files, config files (`.toml`, `.yaml`, `.json`), documentation (`.md`, `.txt`).

## 📋 Pre-Commit Validation Checklist

Before suggesting any code changes, verify:

- [ ] `mypy src/ tests/` passes with no errors
- [ ] All functions have complete type annotations
- [ ] No direct OS imports in `src/` code (use adaptors)
- [ ] `pytest --cov=src/dd_license_attribution --cov-fail-under=95` passes
- [ ] New code has corresponding unit tests with proper mocking
- [ ] Unit tests mock all external dependencies (adaptors, network calls, etc.)
- [ ] All mocks are verified with assertions (call count AND parameters)
- [ ] No unit tests make real filesystem, network, or database calls
- [ ] Tests focus on public interfaces, not internal implementation details
- [ ] `isort --check-only src/ tests/` and `black --check src/ tests/` pass
- [ ] No unused imports remain in the code
- [ ] CHANGELOG.md updated for ALL user-facing changes
- [ ] Modern Python 3.11+ syntax used for type hints (e.g., `list[str]` not `List[str]`)
- [ ] Logging follows consistent format and patterns
- [ ] Contract tests added for any new external library dependencies
- [ ] Full end-to-end CLI validation run completed for new user-facing features
- [ ] All Python files include SPDX-License-Identifier header on first line

## 🔧 Type Safety Requirements

Use modern built-in generics (PEP 585). Import from `typing` only for `Any`, `Protocol`, `TypeAlias`, `Literal`, `TypeVar`, `Generic`.

```python
# ✅ REQUIRED
def process(data: list[str], config: dict[str, Any]) -> str | None: ...

# ❌ FORBIDDEN
from typing import Dict, List, Optional
def process(data: List[str], config: Dict[str, Any]) -> Optional[str]: ...
```

## 🔌 OS Operations Through Adaptors

### FORBIDDEN Direct OS Imports

Never import these modules directly in `src/` code:

```python
# ❌ FORBIDDEN in src/ code
import os
import sys
import subprocess
import pathlib
import shutil
import tempfile
```

### CLI and Infrastructure Exceptions

These are **permitted** in specific contexts only:

1. **`print()` for STDOUT** — allowed in CLI command functions for output intended for piping/redirection; not for debug messages.
2. **`sys.exit()`** — allowed in CLI command functions; not in business logic (raise exceptions instead).
3. **`sys.stderr`** — allowed in `utils/logging.py` for configuring log handlers only.

### REQUIRED Adaptor Usage

```python
from dd_license_attribution.adaptors.os import OSAdaptor

class FileProcessor:
    def __init__(self, os_adaptor: OSAdaptor) -> None:
        self.os_adaptor = os_adaptor

    def process_file(self, path: str) -> str:
        if self.os_adaptor.path_exists(path):
            return self.os_adaptor.read_file(path)
        return ""
```

### Command Execution Rules

All external command execution MUST go through adaptor functions in `dd_license_attribution.adaptors.os`. Use **argument lists**, not shell strings.

```python
# ✅ REQUIRED
run_command(["git", "clone", "--depth", "1", url, path])
output = output_from_command(["go", "list", "-json", "all"], cwd=project_path, env={"GOTOOLCHAIN": "auto"})

# ❌ FORBIDDEN
subprocess.run(command, shell=True)
os.system(f"git clone {url}")
```

- Use `cwd=` instead of `cd path &&` shell patterns
- Use `env=` instead of `VAR=value` shell prefixes (env is merged with the current environment)
- Use Python string operations instead of shell pipes — e.g., `if ref in output` instead of piping to `grep`

### Creating New Adaptors

When OS functionality is needed that doesn't exist, define a Protocol and both real and mock implementations:

```python
from typing import Protocol

class NetworkAdaptor(Protocol):
    def make_request(self, url: str) -> str: ...

class RealNetworkAdaptor:
    def make_request(self, url: str) -> str:
        return requests.get(url).text
```

## 🧪 Testing Requirements

### Coverage Targets

- **Core business logic**: 100% line coverage, 90% branch coverage
- **CLI interfaces**: 100% line coverage, 85% branch coverage
- **Adaptors**: Not unit tested (simple wrappers)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataDog/dd-license-attribution](https://github.com/DataDog/dd-license-attribution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
