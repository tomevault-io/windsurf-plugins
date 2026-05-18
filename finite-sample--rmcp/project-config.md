---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
RMCP (R Model Context Protocol) is a statistical analysis server that bridges AI assistants with R statistical computing capabilities through the Model Context Protocol.

## Development Commands

### Setup & Running (Hybrid Approach)

**For Python-only development (cross-platform):**
```bash
uv sync --group dev              # Install minimal Python dependencies
uv run rmcp start                # Start server in stdio mode (no R tools)
uv run rmcp serve-http           # Start HTTP server with SSE

# Use configuration options
uv run rmcp --debug start        # Enable debug mode
uv run rmcp --config config.json start  # Use custom config file
RMCP_LOG_LEVEL=DEBUG uv run rmcp start  # Use environment variables
```

**For R integration development (Docker-based):**
```bash
docker build -f docker/Dockerfile --target development -t rmcp-dev .  # Build R + Python dev environment
docker run -v $(pwd):/workspace -it rmcp-dev bash
# Inside container:
cd /workspace && pip install -e .
rmcp start                        # Full R integration available
```

**For production deployment (optimized):**
```bash
docker build -f docker/Dockerfile --target production -t rmcp-production .  # Multi-stage optimized build
docker run -p 8000:8000 rmcp-production rmcp http                          # Production HTTP server
```

### Testing (Hybrid Strategy)

**Python-only tests (cross-platform via uv):**
```bash
uv run pytest tests/unit/        # Schema validation, JSON-RPC, transport
uv run ruff check .              # Linting and import sorting
uv run ruff format --check .     # Code formatting check
```

**Complete integration tests (Docker-based):**
```bash
# Docker includes R + FastAPI for complete test coverage (all 240+ tests)
docker run -v $(pwd):/workspace rmcp-dev bash -c "cd /workspace && pip install -e . && pytest tests/integration/"
docker run -v $(pwd):/workspace rmcp-dev bash -c "cd /workspace && pip install -e . && pytest tests/scenarios/"
# Full test suite: smoke + unit + integration + scenarios + protocol + config
docker run -v $(pwd):/workspace rmcp-dev bash -c "cd /workspace && pip install -e . && pytest tests/"
```

### Code Quality
```bash
# Python formatting (cross-platform)
uv run ruff check --fix .         # Auto-fix linting issues and sort imports
uv run ruff format .              # Format code
uv run mypy rmcp

# R formatting (Docker-based)
docker run -v $(pwd):/workspace rmcp-dev R -e "library(styler); style_file(list.files('rmcp/r_assets', pattern='[.]R$', recursive=TRUE, full.names=TRUE))"

# Documentation building (cross-platform)
uv run sphinx-build docs docs/_build       # Build documentation
uv run sphinx-build -b html docs docs/_build/html  # HTML output
uv run sphinx-autogen docs/**/*.rst        # Generate autosummary stubs
```

## Architecture

### Core Components
- **Transport Layer** (`rmcp/transport/`): Handles stdio and HTTP+SSE communication
- **Core Server** (`rmcp/core/`): MCPServer, Context management, JSON-RPC protocol
- **Registries** (`rmcp/registries/`): Dynamic registration for tools, resources, prompts
- **Tools** (`rmcp/tools/`): 53 statistical analysis tools across 11 categories
- **Comprehensive Package Whitelist**: 429 R packages from CRAN task views with tiered security
- **R Integration** (`rmcp/r_integration.py`): Python-R bridge via subprocess + JSON

### Adding New Statistical Tools
1. Create tool function in appropriate file under `rmcp/tools/`
2. Use `@tool` decorator with input/output JSON schemas
3. Implement corresponding R script in `r_assets/scripts/`
4. Write schema validation tests in `tests/unit/tools/` (Python-only)
5. Write functional tests in `tests/integration/tools/` (real R execution)
6. Add scenario tests in `tests/scenarios/` for user workflows

### Key Design Patterns
- **Registry Pattern**: All tools/resources/prompts use centralized registries
- **Context Pattern**: Request-scoped state management with lifecycle
- **Transport Abstraction**: Common interface for stdio/HTTP transports
- **Virtual Filesystem**: Security sandbox for file operations (`rmcp/security/vfs.py`)
- **Universal Operation Approval**: User consent system for file operations and package installation

### Testing Strategy (Optimized Organization)

**Tier 1: Basic Functionality (Python-only, cross-platform)**
- **Smoke tests** (`tests/smoke/`): Basic server startup, CLI, imports (no R required, very fast)
- **Unit tests** (`tests/unit/`): Pure Python logic organized by component:
  - `tests/unit/core/`: Server, context, schemas
  - `tests/unit/tools/`: Tool schema validation
  - `tests/unit/transport/`: HTTP transport logic

**Tier 2: Integration Testing (R + FastAPI required, Docker-based)**
- **Protocol tests** (`tests/integration/protocol/`): MCP protocol validation with mocked R responses
- **Tool integration** (`tests/integration/tools/`): Real R execution for statistical tool functionality
- **Transport integration** (`tests/integration/transport/`): HTTP transport with real FastAPI server
- **Core integration** (`tests/integration/core/`): Server registries, capabilities, error handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [finite-sample/rmcp](https://github.com/finite-sample/rmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
