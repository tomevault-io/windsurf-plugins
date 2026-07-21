---
trigger: always_on
description: Guidelines for AI coding agents (Copilot, Cursor, Claude Code, etc.) working in this repository.
---

# AGENTS.md

Guidelines for AI coding agents (Copilot, Cursor, Claude Code, etc.) working in this repository.

## Project Overview

LMCache is a KV cache management engine for LLM serving that reduces Time To First Token (TTFT) and increases throughput. It stores KV caches across multiple tiers (GPU, CPU, disk, S3) and integrates with vLLM and SGLang.

## Repository

The default branch is `dev`. Base all new branches and pull requests against `dev`.

## Python Environment

We recommend using [uv](https://docs.astral.sh/uv/) to manage Python environments and dependencies:

```bash
# Create and activate a virtual environment
uv venv --python 3.12
source .venv/bin/activate

# Install dependencies
uv pip install torch               # pre-requisite for CUDA extensions
uv pip install -e . --no-build-isolation
```

## Build & Install

```bash
# Standard install with CUDA extensions (requires torch pre-installed)
pip install -e . --no-build-isolation

# Source-only (no native extensions)
NO_NATIVE_EXT=1 pip install -e .

# CPU-only (common C++ extensions, no GPU backend)
NO_GPU_EXT=1 pip install -e . --no-build-isolation

# HIP/ROCm build
BUILD_WITH_HIP=1 pip install -e .
```

## Testing

### Running Tests

```bash
# Run standard test suite (mirrors CI)
pytest -xvs --ignore=tests/disagg \
 --ignore=tests/v1/multiprocess/ \
 --ignore=tests/v1/distributed/ \
 --ignore=tests/skipped \
 --ignore=tests/v1/storage_backend/test_eic.py

# Run a single test file
pytest -xvs tests/v1/test_cache_engine.py

# Run a single test
pytest -xvs tests/v1/test_cache_engine.py::test_function_name
```

Test dependencies: `uv pip install -r requirements/test.txt`

Pytest marker: `@pytest.mark.no_shared_allocator` disables the shared-allocator monkeypatch for a test.

### Testing Practices

- Write tests against the **public interface and docstring contract**, not the implementation. Test as if you don't know the internals — verify that behavior matches what the docstring describes.
- Avoid accessing private members in tests unless strongly needed.
- All new features and bug fixes should include corresponding tests.
- Ensure existing tests still pass before submitting changes.

## Linting & Code Quality

```bash
# Run all checks (mirrors CI exactly)
pre-commit run --all-files

# Individual tools
ruff check .              # Lint (E, F, B, SLF rules)
ruff format .             # Format (line-length 88)
isort .                   # Import sorting (black profile, from_first=true)
mypy --config-file=pyproject.toml   # Type checking
codespell --toml pyproject.toml     # Spell checking
```

C++/CUDA files use clang-format (Google style, 80-col). Rust code in `rust/` uses `cargo fmt` and `cargo clippy`.

All Python files require an `# SPDX-License-Identifier: Apache-2.0` header as the first line.

### Import Ordering

Imports must follow this section-heading convention:

```python
# Standard
import os

# Third Party
import torch

# First Party
from lmcache.v1.config import LMCacheEngineConfig

# Local
from .utils import helper
```

### SLF (Private Member Access)

SLF lint rules are currently enforced by CI only in `lmcache/v1/multiprocess/` and `lmcache/v1/distributed/`. However, **all new code should follow SLF discipline regardless of location** — never access private members (prefixed with `_`) of other classes. Treat this as a project-wide coding standard for any new or modified code.

## Coding Conventions

### Type Hints

All functions and methods must have type hints for their arguments and return values.

### Docstrings

Every public function and method must have a clear docstring covering:
- What the function does
- Arguments (with types and descriptions)
- Return values
- Raised exceptions (if any)
- Additional notes when behavior is non-obvious

### Writing Documentation

LMCache has three documentation surfaces:

1. **User-facing docs** (`docs/source/`, reStructuredText, Sphinx-built). When adding
   or modifying user docs, place them in the appropriate subdirectory under
   `docs/source/` (e.g., `developer_guide/`, `getting_started/`, `kv_cache/`) and link
   new pages from a `toctree` so they appear in the built site.
2. **Design docs** (`docs/design/`, Markdown). **`docs/design/` mirrors the `lmcache/`
   package tree** — a design doc for `lmcache/<path>/` lives at `docs/design/<path>/`.
   For example, `lmcache/v1/distributed/l2_adapters/` → `docs/design/v1/distributed/l2_adapters/`.
   When adding a design doc, place it at the path matching the module it describes;
   when touching existing docs, find them at the mirrored location. See
   `docs/design/README.md` for the full convention.
3. **Module READMEs** (`README.md` next to code). Stay in place as user-entry-points;
   they are symlinked from `docs/design/<path>/README.md`. Do not move them.

When writing or updating documentation, follow these principles:

- **Be concrete and concise.** State exactly what something does and why — avoid vague, hand-wavy descriptions. One precise sentence beats a paragraph of generalities.
- **Include examples.** Show concrete code snippets, command invocations, or data formats so the reader can immediately see how things work in practice.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LMCache/LMCache](https://github.com/LMCache/LMCache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
