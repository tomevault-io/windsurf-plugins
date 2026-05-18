---
trigger: always_on
description: At the start of your every response, say "APPLIED MODELCONTEXTPROTOCOL RULE".
---

At the start of your every response, say "APPLIED MODELCONTEXTPROTOCOL RULE".

# Model Context Protocol (MCP) Server Development Rules and Guidelines

## Overview

This document provides comprehensive rules, guidelines, and best practices for developing Model Context Protocol (MCP) servers, based on the official MCP specification and the Apache Airflow MCP server implementation.

## Core MCP Concepts

### 1. MCP Server Capabilities

MCP servers can provide three main types of capabilities:

1. **Resources**: File-like data that can be read by clients (like API responses or file contents)
2. **Tools**: Functions that can be called by the LLM (with user approval)
3. **Prompts**: Pre-written templates that help users accomplish specific tasks

### 2. Architecture Principles

- **Stateless Design**: Servers should be stateless and handle each request independently
- **Standard Transport**: Use stdio or SSE transport protocols as defined in the MCP specification
- **Type Safety**: Leverage Python type hints for automatic tool definition generation
- **Async First**: Use asynchronous programming patterns for better performance

## Development Guidelines

### 1. Project Structure

Follow this recommended project structure:

```
project-root/
├── src/
│   ├── __init__.py
│   ├── __main__.py
│   ├── main.py              # CLI entry point
│   ├── server.py            # FastMCP server instance
│   ├── enums.py             # Project enums
│   ├── envs.py              # Environment configuration
│   └── [domain]/            # Domain-specific modules
│       ├── __init__.py
│       ├── client.py        # External API client setup
│       └── [feature].py     # Feature-specific tool implementations
├── pyproject.toml
├── README.md
├── Makefile
└── @modelcontextprotocol.mdc  # This file
```

### 2. Tool Implementation Standards

#### Tool Function Signature

All tool functions must follow this pattern:

```python
from typing import Any, List, Optional, Union
import mcp.types as types

async def tool_name(
    required_param: str,
    optional_param: Optional[str] = None
) -> List[Union[types.TextContent, types.ImageContent, types.EmbeddedResource]]:
    """
    Brief description of what the tool does.
    
    Args:
        required_param: Description of required parameter
        optional_param: Description of optional parameter
    
    Returns:
        List of MCP content types
    """
    # Implementation here
    return [types.TextContent(type="text", text=str(result))]
```

#### Tool Registration Pattern

Use this pattern in each feature module:

```python
def get_all_functions() -> list[tuple[Callable, str, str]]:
    """Return list of (function, name, description) tuples for registration."""
    return [
        (function_name, "tool_name", "Tool description"),
        # Add more tools here
    ]
```

### 3. Configuration Management

#### Environment Variables

- Use descriptive environment variable names with consistent prefixes
- Provide sensible defaults where appropriate
- Document all required environment variables in README
- Validate environment variables at startup

```python
import os
from urllib.parse import urlparse

# Example from envs.py
SERVICE_HOST = urlparse(os.getenv("SERVICE_HOST"))._replace(path="").geturl().rstrip("/")
SERVICE_USERNAME = os.getenv("SERVICE_USERNAME")
SERVICE_PASSWORD = os.getenv("SERVICE_PASSWORD")
SERVICE_API_VERSION = os.getenv("SERVICE_API_VERSION", "v1")
```

#### Client Configuration

- Centralize external API client configuration
- Use configuration objects for complex setups
- Handle authentication securely

```python
from external_client import ApiClient, Configuration

configuration = Configuration(
    host=urljoin(SERVICE_HOST, f"/api/{SERVICE_API_VERSION}"),
    username=SERVICE_USERNAME,
    password=SERVICE_PASSWORD,
)
api_client = ApiClient(configuration)
```

### 4. Error Handling

#### Graceful Degradation

- Handle API failures gracefully
- Provide meaningful error messages
- Use try-catch blocks for external API calls
- Return informative error responses

```python
async def resilient_tool():
    try:
        response = await external_api.call()
        return [types.TextContent(type="text", text=str(response))]
    except ExternalAPIError as e:
        error_msg = f"Failed to fetch data: {str(e)}"
        return [types.TextContent(type="text", text=error_msg)]
```

#### Validation

- Validate input parameters
- Check required environment variables at startup
- Provide clear validation error messages

### 5. Response Format Standards

#### Consistent Response Structure

- Always return List[Union[types.TextContent, types.ImageContent, types.EmbeddedResource]]
- Convert API responses to dictionaries for easier manipulation
- Add metadata when helpful (e.g., UI URLs, timestamps)

```python
async def get_resource(resource_id: str):
    response = api.get_resource(resource_id)
    response_dict = response.to_dict()
    
    # Add helpful metadata
    response_dict["ui_url"] = get_resource_url(resource_id)
    response_dict["fetched_at"] = datetime.utcnow().isoformat()
    
    return [types.TextContent(type="text", text=str(response_dict))]
```

### 6. CLI Interface Standards

#### Click Integration

Use Click for command-line interfaces:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yangkyeongmo/mcp-server-apache-airflow](https://github.com/yangkyeongmo/mcp-server-apache-airflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
