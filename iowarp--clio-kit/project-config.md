---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**CLIO Kit** is part of the IoWarp platform's tooling layer for AI agents. It is a production-grade monorepo containing 15+ MCP (Model Context Protocol) servers designed for scientific computing research, with plans to expand to additional agent capabilities (skills, plugins, extensions). The project enables AI agents and LLMs to interact with HPC resources, scientific data formats, and research datasets through a standardized protocol.

The repository uses a **unified launcher with auto-discovery** pattern: each MCP server is independently developed and tested, but all are launched through a single `clio-kit <server-name>` command.

**Platform Context**: CLIO Kit is the tooling layer of the IoWarp platform, providing comprehensive agent capabilities beyond just MCP servers.

**Key Technologies**: FastMCP 3.0, Python 3.10+, UV package manager, Pydantic, pytest, Ruff

## Project Structure

```
clio-kit/                           # Monorepo root
├── src/clio_kit/                   # Unified launcher CLI
├── clio-kit-mcp-servers/                # 16 independent MCP servers
│   ├── hdf5/ ⭐                       # Flagship server (v2.0, 28 tools)
│   ├── pandas/                        # Data analysis operations
│   ├── slurm/                         # HPC job management
│   ├── arxiv/                         # Research paper fetching
│   ├── chronolog/                     # Distributed logging
│   ├── compression/                   # File compression
│   ├── darshan/                       # I/O performance analysis
│   ├── jarvis/                        # Data pipeline management
│   ├── lmod/                          # Environment modules
│   ├── ndp/                           # Dataset discovery
│   ├── node-hardware/                 # System hardware info
│   ├── parallel-sort/                 # Large file sorting
│   ├── paraview/                      # Scientific visualization
│   ├── parquet/                       # Parquet file handling
│   ├── plot/                          # Data visualization
│   ├── adios/                         # ADIOS2 data I/O
│   └── [each has its own pyproject.toml, dependencies, tests]
├── clio-agentic-search/             # Standalone hybrid retrieval engine (not an MCP server)
├── clio-kit-website/                # Docusaurus documentation site
├── scripts/                           # Utility scripts (generate_docs.py, etc)
├── .github/workflows/                 # CI/CD automation
└── [root config files]                # pyproject.toml, uv.lock

```

**Key Design Pattern:**
- Root `pyproject.toml` only includes launcher dependencies (click)
- Each MCP server in `clio-kit-mcp-servers/` is a complete Python package with its own `pyproject.toml`, entry point, and isolated dependencies
- Launcher auto-discovers servers by scanning for `pyproject.toml` files
- Servers run via `uvx` with isolated environments for dependency isolation

## Common Development Commands

### Setup & Installation

```bash
# Install all dependencies (development mode)
uv sync --all-extras --dev

# For a specific MCP server
cd clio-kit-mcp-servers/hdf5
uv sync --all-extras --dev
```

### Code Quality & Testing

#### Run all quality checks (mimics CI):
```bash
# For a specific MCP server
cd clio-kit-mcp-servers/hdf5

# Ruff: Linting + formatting
uv run ruff check .
uv run ruff format . --check

# MyPy: Type checking
uv run mypy src/

# pytest: Tests with coverage
uv run pytest -v --cov=src/

# pip-audit: Security vulnerabilities
uv run pip-audit

# FastMCP 3.0 validation (instructions, annotations, tags, resources, prompts)
uv run python ../../scripts/validate_fastmcp.py
```

#### Quick test runs:
```bash
# Run all tests in a server
cd clio-kit-mcp-servers/hdf5
uv run pytest -v

# Run a single test file
uv run pytest tests/test_server.py -v

# Run a specific test
uv run pytest tests/test_server.py::TestClass::test_method -v

# Run with coverage report
uv run pytest --cov=src/ --cov-report=html
```

### Running Servers

```bash
# Via launcher (from root directory)
uvx clio-kit hdf5

# Direct development mode (from server directory)
cd clio-kit-mcp-servers/hdf5
uv run hdf5-mcp

# List all available MCPs
uvx clio-kit
```

### Code Formatting & Fixing

```bash
# Format code automatically (Ruff)
cd clio-kit-mcp-servers/hdf5
uv run ruff format .

# Fix linting issues automatically
uv run ruff check --fix .

# Verify types after changes
uv run mypy src/
```

### clio-agentic-search (Standalone Service)

```bash
# Setup
cd clio-agentic-search
uv sync --all-extras --dev

# Quality checks
uv run ruff check .
uv run ruff format --check .
uv run mypy src/
uv run pytest --ignore=tests/benchmarks -v

# Full quality gate (6 checks)
uv run python -m clio_agentic_search.evals.quality_gate

# Run API server
uv run uvicorn clio_agentic_search.api.app:app --reload

# CLI
uv run clio query --namespace local_fs --q "pressure between 190 and 360 kPa"
uv run clio index --namespace local_fs
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iowarp/clio-kit](https://github.com/iowarp/clio-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
