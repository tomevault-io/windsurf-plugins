---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

IntelliKit is a monorepo of LLM-ready GPU profiling and analysis tools for AMD ROCm. It provides clean Python abstractions over complex GPU internals with MCP (Model Context Protocol) server support for LLM integration.

**Requirements:** The repo-level `install/tools/install.sh` script enforces Python >= 3.10, but individual packages have lower minimums: `accordo`, `linex`, and `nexus` require Python >= 3.8; `metrix` requires Python >= 3.9; `kerncap`, `rocm_mcp`, and `uprof_mcp` require Python >= 3.10. ROCm >= 6.0 is the general baseline; kerncap and linex target ROCm 7.0+ workflows. MI300+ GPUs are needed for the full profiling stack, while RDNA support (gfx1151/gfx1201) is available in metrix.

## Tool Descriptions

| Tool | Purpose | Key Use Case |
| ------ | --------- | -------------- |
| **accordo** | Kernel validation | Verify optimized GPU kernels match reference implementations (CLI + MCP) |
| **kerncap** | Kernel extraction | Isolate and capture GPU kernel dispatches for standalone reproducers (HIP, Triton) |
| **linex** | Source-line profiling | Map cycle-level timing and stall analysis to source code lines (MCP-only) |
| **metrix** | Hardware counter metrics | Profile GPU kernels with human-readable performance insights (CLI + MCP) |
| **nexus** | HSA packet interception | Capture GPU kernel launches and memory operations (MCP-only) |
| **rocm_mcp** | ROCm MCP servers | LLM-accessible HIP compilation, docs, system info, and GPU management (amd-smi) |
| **uprof_mcp** | uProf MCP server | LLM-accessible AMD uProf CPU profiling |

## Build Commands

```bash
# Install all tools from Git (supported path for users and CI-style setups)
# Default pip command is pip3; script requires Python 3.10+ (checks before installing).
curl -sSL https://raw.githubusercontent.com/AMDResearch/intellikit/main/install/tools/install.sh | bash
# Subset only: ... | bash -s -- --tools metrix,linex
# From a clone:
#   ./install/tools/install.sh [--tools ...] [--pip-cmd ...] [--repo-url ...] [--ref ...] [--dry-run]

# Editable installs for development (any subset; from repo root)
pip install -e accordo/ -e kerncap/ -e linex/ -e metrix/ -e nexus/ -e rocm_mcp/ -e uprof_mcp/

# Install individual tools
pip install -e metrix/
pip install -e linex/

# Build nexus C++ component (scikit-build-core handles CMake automatically)
pip install -e nexus/
# Or build manually if needed:
# cd nexus && mkdir -p build && cd build && cmake .. && make

# Build kerncap (scikit-build-core builds libkerncap.so + kerncap-replay)
pip install -e kerncap/
```

## Testing

Most tools have pytest-based test suites under `tests/`:

- **accordo**: `tests/` directory
- **kerncap**: `tests/unit/` and `tests/integration/` with pytest markers (`docker`, `gpu`)
- **linex**: `tests/` directory
- **metrix**: `tests/unit/` and `tests/integration/` with pytest markers (`unit`, `integration`, `e2e`, `slow`)
- **nexus**: `tests/` directory
- **rocm_mcp**: `tests/` directory
- **uprof_mcp**: no `tests/` directory yet; rely on examples/manual validation

All tools also have `examples/` directories for usage demonstrations.

```bash
# Run tests for individual tools
cd metrix && pytest
cd accordo && pytest
cd nexus && pytest
cd linex && pytest

# Run specific test file
pytest metrix/tests/unit/test_api.py
pytest accordo/tests/test_mcp_server.py
pytest kerncap/tests/unit/test_mcp_server.py

# Run metrix tests by marker (defined in metrix/pytest.ini)
pytest -m unit      # Fast unit tests
pytest -m integration  # Requires GPU/rocprof
pytest -m e2e      # End-to-end tests (require GPU and benchmarks)
pytest -m slow     # Slow tests (> 5s)

# Run kerncap unit tests (no GPU required)
cd kerncap && pytest tests/unit/

# Run MCP entrypoint tests (no GPU required; CLI transport parsing only)
pytest accordo/tests/test_mcp_server.py
pytest kerncap/tests/unit/test_mcp_server.py

# Run rocm_mcp tests
cd rocm_mcp && pytest tests/

# uprof_mcp currently has examples but no pytest suite
```

## Linting

```bash
# Lint entire repo (shared config: ruff.toml; packages may extend and override)
ruff check .
ruff format .

# Lint specific tool
ruff check metrix/
```

## Architecture

### Monorepo Structure

Each tool is a standalone Python package with its own `pyproject.toml`:

| Tool | Build System | Description |
| ------ | -------------- | ------------- |
| **accordo** | scikit-build-core (CMake), setuptools-scm | GPU kernel validation, C++ compiled at runtime, CLI tool |
| **kerncap** | scikit-build-core (CMake), setuptools-scm | Kernel extraction and isolation, C++ HSA interception, CLI tool |
| **linex** | setuptools, setuptools-scm | Source-level SQTT profiling (`src/` layout), MCP-only |
| **metrix** | setuptools, setuptools-scm | Hardware counter profiling (`src/` layout), CLI + MCP, RDNA support |
| **nexus** | scikit-build-core (CMake), setuptools-scm | HSA packet interception, C++ shared library, MCP-only |
| **rocm_mcp** | setuptools | MCP servers for ROCm tools (`src/` layout) |
| **uprof_mcp** | setuptools | MCP server for AMD uProf CPU profiling (`src/` layout) |

### Metrix Backend System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AMDResearch/intellikit](https://github.com/AMDResearch/intellikit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
