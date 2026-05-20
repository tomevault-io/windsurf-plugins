---
trigger: always_on
description: This guide helps AI assistants implement and modify the MCP Azure DevOps server codebase effectively.
---

# MCP Azure DevOps Server Guide

This guide helps AI assistants implement and modify the MCP Azure DevOps server codebase effectively.

## 1. Purpose & Overview

This MCP server enables AI assistants to interact with Azure DevOps by:
- Connecting to Azure DevOps services via REST API and Python SDK
- Exposing Azure DevOps data (work items, repositories, pipelines, PRs)
- Providing tools to create and modify Azure DevOps objects
- Including prompts for common workflows
- Using PAT authentication for secure interactions

## 2. Project Structure

```
mcp-azure-devops/
├── docs/                      # API documentation
├── src/                       # Source code
│   └── mcp_azure_devops/      # Main package
│       ├── features/          # Feature modules
│       │   ├── projects/      # Project management features
│       │   ├── teams/         # Team management features 
│       │   └── work_items/    # Work item management features
│       │       ├── tools/     # Work item operation tools
│       │       ├── common.py  # Common utilities for work items
│       │       └── formatting.py # Formatting helpers
│       ├── utils/             # Shared utilities
│       ├── __init__.py        # Package initialization
│       └── server.py          # Main MCP server
├── tests/                     # Test suite
├── .env                       # Environment variables (not in repo)
├── CLAUDE.md                  # AI assistant guide
├── LICENSE                    # MIT License
├── pyproject.toml             # Project configuration
├── README.md                  # Project documentation
└── uv.lock                    # Package dependency locks
```

## 3. Core Concepts

### Azure DevOps & MCP Integration

This project bridges two systems:

1. **Azure DevOps Objects**:
   - Work items (bugs, tasks, user stories, epics)
   - Repositories and branches
   - Pull requests and code reviews
   - Pipelines (build and release)
   - Projects and teams

2. **MCP Components**:
   - **Tools**: Action performers that modify data (like POST/PUT/DELETE endpoints)
   - **Prompts**: Templates for guiding interactions

### Authentication

The project requires these environment variables:
- `AZURE_DEVOPS_PAT`: Personal Access Token with appropriate permissions
- `AZURE_DEVOPS_ORGANIZATION_URL`: The full URL to your Azure DevOps organization

## 4. Implementation Guidelines

### Feature Structure

Each feature in the `features/` directory follows this pattern:
- `__init__.py`: Contains `register()` function to add the feature to the MCP server
- `common.py`: Shared utilities, exceptions, and helper functions
- `tools.py` or `tools/`: Functions or classes for operations (GET, POST, PUT, DELETE)

### Tool Implementation Pattern

When implementing a new tool:

1. Define a private implementation function with `_name_impl` that takes explicit client objects:
```python
def _get_data_impl(client, param1, param2):
    # Implementation
    return formatted_result
```

2. Create a public MCP tool function that handles client initialization and error handling:
```python
@mcp.tool()
def get_data(param1, param2):
    """
    Docstring following the standard pattern.
    
    Use this tool when you need to:
    - First use case
    - Second use case
    
    Args:
        param1: Description
        param2: Description
        
    Returns:
        Description of the return value format
    """
    try:
        client = get_client()
        return _get_data_impl(client, param1, param2)
    except AzureDevOpsClientError as e:
        return f"Error: {str(e)}"
```

3. Register the tool in the feature's `__init__.py` or `register_tools()` function

### Function Docstring Pattern

All public tools must have detailed docstrings following this pattern:

```python
"""
Brief description of what the tool does.

Use this tool when you need to:
- First use case with specific example
- Second use case with specific example
- Third use case with specific example

IMPORTANT: Any special considerations or warnings.

Args:
    param1: Description of first parameter
    param2: Description of second parameter
    
Returns:
    Detailed description of what is returned and in what format
"""
```

### Error Handling

The standard error handling pattern is:

```python
try:
    # Implementation code
except AzureDevOpsClientError as e:
    return f"Error: {str(e)}"
except Exception as e:
    return f"Error doing operation: {str(e)}"
```

For specific errors, create custom exception classes in the feature's `common.py` file.

## 5. Common Code Patterns

### Client Initialization

```python
from mcp_azure_devops.utils.azure_client import get_connection

def get_work_item_client():
    """Get the Work Item Tracking client."""
    try:
        connection = get_connection()
        return connection.clients.get_work_item_tracking_client()
    except Exception as e:
        raise AzureDevOpsClientError(f"Failed to get Work Item client: {str(e)}")
```

### Response Formatting

```python
def format_result(data):
    """Format data for response."""
    formatted_info = [f"# {data.name}"]
    
    # Add additional fields with null checks
    if hasattr(data, "description") and data.description:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vortiago/mcp-azure-devops](https://github.com/Vortiago/mcp-azure-devops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
