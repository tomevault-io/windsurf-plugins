---
trigger: always_on
description: This file provides guidance for AI coding agents working with this codebase.
---

# AGENTS.md

This file provides guidance for AI coding agents working with this codebase.

## Project Overview

Log-Analyzer-with-MCP is an MCP (Model Context Protocol) server providing AI assistants access to AWS CloudWatch Logs for searching, analysis, and cross-service correlation.

**Tech Stack**: Python 3.12+, uv (package manager), FastMCP, boto3, ruff

## Build & Development Commands

```bash
# Installation
uv sync                      # Install all dependencies
source .venv/bin/activate    # Activate virtual environment

# Running the Server (local development)
python -m cw_mcp_server.server [--profile PROFILE] [--region REGION] [--stateless]
uv run python -m cw_mcp_server.server  # Alternative

# Via uvx (recommended for users)
uvx --from git+https://github.com/awslabs/Log-Analyzer-with-MCP cw-mcp-server [--profile PROFILE] [--region REGION] [--stateless]

# Linting & Formatting (run before committing)
pre-commit run --all-files   # Preferred - runs ruff lint and format
ruff check --fix .           # Lint with auto-fix
ruff format .                # Format code

# Manual Testing (no pytest configured yet)
python src/client.py list-groups [--prefix PREFIX]
python src/client.py search LOG_GROUP "query" [--hours N]
python src/client.py find-errors LOG_GROUP
```

## Code Style Guidelines

### File Headers
All Python files must start with the Apache-2.0 license header:
```python
#!/usr/bin/env python3

# Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
# SPDX-License-Identifier: Apache-2.0
```

### Import Order (ruff-enforced)
1. Standard library imports
2. Third-party imports (boto3, mcp, etc.)
3. Local imports (relative imports with `.`)

```python
import asyncio
import json
from typing import List, Optional, Callable

import boto3
from mcp.server.fastmcp import FastMCP

from .tools.search_tools import CloudWatchLogsSearchTools
```

### Type Hints
Use type hints consistently. For Python 3.12+, prefer modern syntax but the codebase currently uses `typing` module imports for compatibility:
```python
async def search_logs(
    self,
    log_group_name: str,
    query: str,
    hours: int = 24,
    start_time: str = None,
    end_time: str = None,
) -> str:
```

### Docstrings
Use Google-style docstrings with Args and Returns:
```python
def get_log_groups(self, prefix: str = None, limit: int = 50) -> str:
    """
    Get a list of CloudWatch Log Groups with optional filtering.

    Args:
        prefix: Optional prefix to filter log groups by name
        limit: Maximum number of log groups to return (default: 50)

    Returns:
        JSON string with log groups information
    """
```

### Naming Conventions
- **Classes**: PascalCase (`CloudWatchLogsSearchTools`)
- **Functions/Methods**: snake_case (`search_logs`, `get_log_groups`)
- **Variables**: snake_case (`log_group_name`, `formatted_results`)
- **Private methods**: prefix with `_` (`_detect_log_format`)

### Error Handling
1. **Tool methods**: Use `@handle_exceptions` decorator for JSON error responses
2. **Resource methods**: Return JSON errors, don't raise exceptions to clients:
```python
except Exception as e:
    return json.dumps({"error": str(e)}, indent=2)
```

### Return Format
All tool/resource methods return JSON strings:
```python
return json.dumps(formatted_results, indent=2)
```

### AWS Client Pattern
Always support optional profile and region parameters:
```python
def __init__(self, profile_name=None, region_name=None):
    session = boto3.Session(profile_name=profile_name, region_name=region_name)
    self.logs_client = session.client("logs")
```

## Architecture

```
src/
├── client.py                           # CLI client for testing
└── cw_mcp_server/
    ├── server.py                       # Main FastMCP server entry point
    ├── resources/
    │   └── cloudwatch_logs_resource.py # CloudWatch Logs as MCP resources
    └── tools/
        ├── __init__.py                 # @handle_exceptions decorator
        ├── utils.py                    # Time range utilities
        ├── search_tools.py             # Log search/query tools
        ├── analysis_tools.py           # Log analysis tools
        └── correlation_tools.py        # Cross-service correlation
```

### Key Decorators
- **@mcp.tool()**: Exposes functions as MCP tools
- **@mcp.resource()**: Exposes data as MCP resources (`logs://groups/{name}`)
- **@mcp.prompt()**: Provides prompt templates
- **@with_aws_config()**: Handles AWS profile/region override per-call
- **@handle_exceptions**: Returns JSON errors instead of raising

### Entry Points
- Main: `cw_mcp_server.server:main()`
- Console script: `cw-mcp-server` (defined in pyproject.toml)

## Pre-commit Hooks

Pre-commit is configured (`.pre-commit-config.yaml`) and runs:
- `ruff check --fix` - Lint with auto-fix
- `ruff format` - Format code

Install hooks locally:
```bash
pre-commit install
```

**Note**: CI/CD via GitHub Actions is not yet configured. Run `pre-commit run --all-files` locally before committing.

See `.pre-commit-config.yaml` for pinned ruff version.

## Debugging

### MCP Inspector
Use the MCP Inspector to interactively test and debug tools:
```bash
npx @modelcontextprotocol/inspector python -m cw_mcp_server.server
```

### Verbose Logging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/Log-Analyzer-with-MCP](https://github.com/awslabs/Log-Analyzer-with-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
