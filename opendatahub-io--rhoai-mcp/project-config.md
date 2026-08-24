---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RHOAI MCP Server is an MCP (Model Context Protocol) server that enables AI agents to interact with Red Hat OpenShift AI (RHOAI) environments. It provides programmatic access to RHOAI features (projects, workbenches, model serving, pipelines, data connections, storage, training) through domain modules, plus workflow prompts that guide agents through multi-step operations.

## Build and Development Commands

```bash
# Setup development environment
uv sync                          # Install package
make dev                         # Alias for setup

# Run the server locally
uv run rhoai-mcp                 # Default (stdio transport)
uv run rhoai-mcp --transport sse # HTTP transport

# Testing
make test                        # All tests
make test-unit                   # Unit tests only (tests/training)
make test-integration            # Integration tests (tests/integration)

# Code quality
make lint                        # ruff check
make format                      # ruff format + fix
make typecheck                   # mypy
make check                       # lint + typecheck

# Container operations
make build                       # Build container image
make run-http                    # Run with SSE transport
make run-stdio                   # Run with stdio transport
make run-dev                     # Debug logging + dangerous ops enabled

# MCP Evaluation (requires LLM API key, see .env.eval.example)
make eval                        # Run mock-cluster evals
make eval-live                   # Run all evals (including live cluster)
make eval-scenario SCENARIO=cluster_exploration  # Run a single scenario
```

## Architecture

### Project Structure

```
rhoai-mcp/
├── src/
│   └── rhoai_mcp/               # Main package
│       ├── __init__.py
│       ├── __main__.py          # CLI entry point
│       ├── config.py            # Configuration (pydantic-settings)
│       ├── server.py            # FastMCP server
│       ├── hooks.py             # Pluggy hook specifications
│       ├── plugin.py            # Plugin protocol and base class
│       ├── plugin_manager.py    # Plugin lifecycle management
│       ├── clients/             # K8s client abstractions
│       ├── models/              # Shared Pydantic models
│       ├── utils/               # Helper functions
│       ├── domains/             # Domain modules (pure CRUD operations)
│       │   ├── projects/        # Data Science Project management
│       │   ├── notebooks/       # Kubeflow Notebook/Workbench
│       │   ├── inference/       # KServe InferenceService
│       │   ├── pipelines/       # Data Science Pipelines (DSPA)
│       │   ├── connections/     # S3 data connections
│       │   ├── storage/         # PersistentVolumeClaim
│       │   ├── training/        # Kubeflow Training Operator
│       │   ├── model_registry/   # Model Registry integration
│       │   ├── prompts/         # MCP workflow prompts (18 prompts)
│       │   └── registry.py      # Domain plugin registry (9 plugins)
│       └── composites/          # Cross-cutting composite tools
│           ├── cluster/         # Cluster summaries and exploration
│           ├── training/        # Training workflow orchestration
│           ├── meta/            # Tool discovery and guidance
│           └── registry.py      # Composite plugin registry (3 plugins)
├── tests/                       # Test suite
├── docs/                        # Documentation
├── pyproject.toml               # Project configuration
└── Containerfile                # Container build
```

**Domains vs Composites**: Domain modules provide CRUD operations for specific Kubernetes resource types. Composite modules provide cross-cutting tools that orchestrate multiple domains (e.g., `prepare_training` validates storage, credentials, and runtime before creating a training job).

### Domain Module Structure

Each domain module in `src/rhoai_mcp/domains/` follows this layout:
```
domains/<name>/
├── __init__.py
├── client.py            # K8s resource client
├── models.py            # Pydantic models
├── tools.py             # MCP tool implementations
├── crds.py              # CRD definitions (if applicable)
├── resources.py         # MCP resources (if applicable)
└── prompts.py           # MCP prompts (if applicable)
```

The domain registry (`domains/registry.py`) defines all domains and provides them to the server for registration.

### Plugin Hooks

Plugins can implement these hooks (defined in `hooks.py`):
- `rhoai_register_tools`: Register MCP tools
- `rhoai_register_resources`: Register MCP resources
- `rhoai_register_prompts`: Register MCP prompts
- `rhoai_get_crd_definitions`: Return CRD definitions
- `rhoai_health_check`: Check plugin health

### Configuration

Environment variables use `RHOAI_MCP_` prefix. Key settings:
- `AUTH_MODE`: auto | kubeconfig
- `TRANSPORT`: stdio | sse | streamable-http
- `KUBECONFIG_PATH`, `KUBECONFIG_CONTEXT`: For kubeconfig auth
- `ENABLE_DANGEROUS_OPERATIONS`: Enable delete operations
- `READ_ONLY_MODE`: Disable all writes

### Key Dependencies


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendatahub-io/rhoai-mcp](https://github.com/opendatahub-io/rhoai-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
