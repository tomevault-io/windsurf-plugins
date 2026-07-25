---
trigger: always_on
description: This guide helps LLM agents understand the Superset MCP (Model Context Protocol) service architecture and development conventions.
---

# MCP Service - LLM Agent Guide

This guide helps LLM agents understand the Superset MCP (Model Context Protocol) service architecture and development conventions.

## CRITICAL: Apache License Headers

**EVERY Python file in the MCP service MUST have the Apache Software Foundation license header.**

This includes:
- All `.py` files (tool files, schemas, __init__.py files, etc.)
- **NEVER remove existing license headers during refactoring or edits**
- **ALWAYS add license headers when creating new files**
- **ALWAYS verify license headers are present after editing files**

If you see a file without a license header, ADD IT IMMEDIATELY. If you accidentally remove one during editing, ADD IT BACK.

Use this exact template at the top of EVERY Python file:

```python
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
```

**Note**: LLM instruction files like `CLAUDE.md`, `AGENTS.md`, etc. are excluded from this requirement (listed in `.rat-excludes`) to avoid token overhead, but ALL other Python files require it.

## Architecture Overview

The MCP service provides programmatic access to Superset via the Model Context Protocol, allowing AI assistants to interact with dashboards, charts, datasets, databases, SQL Lab, and instance metadata.

### Key Components

```
superset/mcp_service/
├── app.py                      # FastMCP app factory and tool registration
├── auth.py                     # Authentication, authorization, and RBAC
├── mcp_config.py              # Default configuration
├── mcp_core.py                # Reusable core classes for tools
├── flask_singleton.py         # Flask app singleton for MCP context
├── middleware.py              # FastMCP middleware (logging, errors, size guards)
├── server.py                  # Server startup (streamable-http, multi-pod)
├── jwt_verifier.py            # JWT token validation
├── chart/                     # Chart tools, schemas, prompts, resources
│   ├── schemas.py
│   ├── chart_utils.py
│   ├── preview_utils.py
│   ├── validation.py
│   ├── tool/
│   ├── prompts/
│   └── resources/
├── dashboard/                 # Dashboard tools and schemas
│   ├── schemas.py
│   └── tool/
├── dataset/                   # Dataset tools and schemas
│   ├── schemas.py
│   └── tool/
├── explore/                   # Explore link generation
│   ├── schemas.py
│   └── tool/
├── sql_lab/                   # SQL Lab tools (execute, save, open)
│   ├── schemas.py
│   └── tool/
├── system/                    # System tools (health, instance info, schema)
│   ├── schemas.py
│   ├── tool/
│   ├── prompts/
│   └── resources/
├── common/                    # Shared error schemas
├── commands/                  # MCP-specific command classes
└── utils/                     # Utilities (URL, schema parsing, error builders)
```

### Dependency Injection Architecture

The `@tool` and `@prompt` decorators are defined as stubs in the `superset-core` package (`superset_core.mcp.decorators`). At startup, `app.py` calls `initialize_core_mcp_dependencies()` which replaces these stubs with concrete implementations that register tools/prompts with the FastMCP instance. This avoids circular imports between `superset_core` and `superset`.

**Startup flow**:
1. `app.py` creates the FastMCP `mcp` instance
2. `initialize_core_mcp_dependencies()` injects the real decorator implementations
3. Tool/prompt/resource imports at the bottom of `app.py` trigger registration
4. `server.py` adds middleware and starts the transport

## Critical Convention: Tool, Prompt, and Resource Registration

**IMPORTANT**: When creating new MCP tools, prompts, or resources, you MUST add their imports to `app.py` for auto-registration. Do NOT add them to `server.py` - that approach doesn't work properly.

### How to Add a New Tool

1. **Create the tool file** in the appropriate directory (e.g., `chart/tool/my_new_tool.py`)
2. **Decorate with `@tool`** using the decorator from `superset_core.mcp.decorators`
3. **Export from the module's `__init__.py`** (e.g., `chart/tool/__init__.py`)
4. **Add import to `app.py`** at the bottom of the file where other tools are imported

**Example (read-only tool)**:
```python
# superset/mcp_service/chart/tool/my_new_tool.py
from fastmcp import Context
from superset_core.mcp.decorators import tool, ToolAnnotations

from superset.extensions import event_logger

@tool(
    tags=["core"],
    class_permission_name="Chart",
    annotations=ToolAnnotations(
        title="My new tool",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/superset](https://github.com/apache/superset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
