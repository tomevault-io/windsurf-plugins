---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Environment Setup:**
```bash
# Always use conda environment instrMCPdev for testing
source ~/miniforge3/etc/profile.d/conda.sh && conda activate instrMCPdev
```

**Package Management:**
```bash
pip install -e .              # Install for development
pip install -e .[dev]         # With dev dependencies
python -m build               # Build package
instrmcp version              # Test installation
```

**Code Quality (CI Requirements - see `.github/workflows/lint.yml`):**
```bash
black --check instrmcp/ tests/                    # Format check (must pass)
flake8 instrmcp/ tests/ --select=E9,F63,F7,F82    # Critical errors only (must pass)
flake8 instrmcp/ tests/ --max-line-length=127     # Style warnings (non-blocking)
mypy instrmcp/ --ignore-missing-imports           # Type check (non-blocking)
```

**Testing:** (Only perform it when explicitly asked)
```bash
# Unit tests (fast, no hardware required - all mocked)
pytest tests/unit/                                  # All unit tests
pytest -v                                           # Verbose
pytest --cov=instrmcp --cov-report=html             # With coverage
pytest tests/unit/test_cache.py                     # Single file
pytest -k "test_cache_initialization"               # Single test by name
pytest tests/unit/test_cache.py::TestReadCache::test_cache_initialization  # Specific test

# E2E tests (requires Playwright and browser automation)
pytest tests/e2e/ -v                                # All E2E tests
pytest tests/e2e/test_01_server_lifecycle.py -v     # Specific E2E test file
pytest tests/e2e/ -v -m p0                          # Priority 0 (critical) tests only

# Playwright tests (metadata consistency checks)
pytest tests/playwright/ -v                         # Playwright-based tests
python tests/playwright/test_metadata_consistency.py --mode snapshot  # Update metadata snapshot
```
Important notes:
- Some tests may stall indefinitely, set a reasonable timeout if needed
- E2E tests launch real JupyterLab servers and use browser automation
- CI excludes E2E and Playwright tests (they run separately or locally)
- Supported Python versions: 3.11, 3.12, 3.13


**CLI Utilities:**
```bash
instrmcp config                           # Show configuration
instrmcp version                          # Show version
```

**Version Management:**
```bash
python tools/version.py              # Show all version locations
python tools/version.py --check      # CI check (exit 1 if mismatch)
python tools/version.py --sync       # Sync all to canonical version
python tools/version.py --bump patch # Bump patch (2.1.0 → 2.1.1)
python tools/version.py --bump minor # Bump minor (2.1.0 → 2.2.0)
python tools/version.py --bump major # Bump major (2.1.0 → 3.0.0)
python tools/version.py --set 2.2.0  # Set specific version
```

**E2E Test Prerequisites:**
```bash
# Install Playwright browsers (one-time setup for E2E tests)
playwright install chromium
```

## Architecture Overview

### Communication Flow
```
Claude Desktop/Code ←→ STDIO ←→ claude_launcher.py ←→ stdio_proxy.py ←→ HTTP ←→ Jupyter MCP Server
                              (agentsetting/claudedesktopsetting/)  (utils/)        (servers/jupyter_qcodes/)

Codex CLI          ←→ STDIO ←→ codex_launcher.py ←→ stdio_proxy.py ←→ HTTP ←→ Jupyter MCP Server
                              (agentsetting/codexsetting/)

Gemini CLI         ←→ STDIO ←→ claude_launcher.py ←→ stdio_proxy.py ←→ HTTP ←→ Jupyter MCP Server
                              (agentsetting/geminisetting/)
```

### Key Directories
- `instrmcp/servers/jupyter_qcodes/` - Main MCP server with QCodes + Jupyter integration
- `instrmcp/servers/jupyter_qcodes/core/` - Always-available tools (qcodes, notebook, resources)
- `instrmcp/servers/jupyter_qcodes/options/` - Optional features (measureit, database, dynamic_tool)
- `instrmcp/utils/stdio_proxy.py` - STDIO↔HTTP proxy for Claude Desktop/Codex
- `instrmcp/extensions/jupyterlab/` - JupyterLab frontend extension
- `instrmcp/cli.py` - Command-line interface
- `tools/version.py` - Unified version management script

### Key Files for Tool Changes
When adding/removing MCP tools, update ALL of these:
1. `instrmcp/servers/jupyter_qcodes/core/` - Core tool implementation
2. `instrmcp/servers/jupyter_qcodes/options/` - Optional feature tools
3. `instrmcp/config/metadata_baseline.yaml` - Add tool/resource descriptions
4. `instrmcp/utils/stdio_proxy.py` - Add/remove tool proxy
5. `docs/ARCHITECTURE.md` - Update tool documentation
6. `README.md` - Update feature documentation

### Metadata Configuration
Tool and resource descriptions are stored in YAML, not hardcoded in Python:
- **Baseline**: `instrmcp/config/metadata_baseline.yaml` (single source of truth)
- **User overrides**: `~/.instrmcp/metadata.yaml` (optional customizations)
- **Config loader**: `instrmcp/utils/metadata_config.py`
- **STDIO client**: `instrmcp/utils/stdio_proxy.py` - `StdioMCPClient` for validation

When adding a new tool, add its description to `metadata_baseline.yaml`:
```yaml
tools:
  my_new_tool:
    title: "My Tool"
    description: |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caidish/instrMCP](https://github.com/caidish/instrMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
