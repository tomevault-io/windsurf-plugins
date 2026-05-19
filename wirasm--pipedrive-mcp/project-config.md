---
trigger: always_on
description: KISS (Keep It Simple, Stupid): Simplicity should be a key goal in design. Choose straightforward solutions over complex ones whenever possible. Simple solutions are easier to understand, maintain, and debug.
---

# CLAUDE.md

KISS (Keep It Simple, Stupid): Simplicity should be a key goal in design. Choose straightforward solutions over complex ones whenever possible. Simple solutions are easier to understand, maintain, and debug.

YAGNI (You Aren't Gonna Need It): Avoid building functionality on speculation. Implement features only when they are needed, not when you anticipate they might be useful in the future.

Dependency Inversion: High-level modules should not depend on low-level modules. Both should depend on abstractions. This principle enables flexibility and testability.

Open/Closed Principle: Software entities should be open for extension but closed for modification. Design your systems so that new functionality can be added with minimal changes to existing code.

IMPORTANT: Before making changes, take time to understand the vertical slice architecture and existing patterns. When solving complex problems, use the phrase "think hard" to activate extended thinking mode for more thorough analysis.

IMPORTANT: NEVER add Claude attribution comment blocks like "Generated with Claude Code" or "Co-Authored-By: Claude" to commit messages or code files. These are unnecessary in this project and will be rejected.

## Project Overview

The mcp-concept project is a Model Control Protocol (MCP) server implementation for interacting with the Pipedrive CRM API. It provides a way for Claude to access and manipulate Pipedrive data through tool calls.

## Environment Setup

1. Create a `.env` file in the root directory with the following environment variables:
   ```
   PIPEDRIVE_API_TOKEN=your_api_token
   PIPEDRIVE_COMPANY_DOMAIN=your_company_domain
   HOST=0.0.0.0  # Optional, defaults to 0.0.0.0
   PORT=8152     # Optional, defaults to 8152
   TRANSPORT=sse  # or "stdio", defaults to "stdio"
   
   # Feature flags (optional)
   PIPEDRIVE_FEATURE_PERSONS=true
   PIPEDRIVE_FEATURE_DEALS=true
   PIPEDRIVE_FEATURE_ORGANIZATIONS=true
   PIPEDRIVE_FEATURE_LEADS=true
   PIPEDRIVE_FEATURE_ITEM_SEARCH=true
   ```

## Dependencies

We always run script and the server with uv run.
`uv run <script>`

This project requires the following dependencies (defined in pyproject.toml):
- httpx >= 0.28.1 (for async HTTP requests)
- mcp[cli] >= 1.8.0 (for MCP server functionality)
- pydantic >= 2.11.4 (for data validation and serialization)
- pytest >= 8.3.5 (for testing)
- pytest-asyncio >= 0.26.0 (for async testing)
- python-dotenv >= 1.1.0 (for environment variable loading)

any additional dependencies should be added by running `uv add <dependency_name>`

## Commands

### Installation

To install the MCP server to Claude desktop:
```bash
cd mcp-concept
mcp install server.py
```

### Running

To run the server locally:
```bash
uv run server.py
```

### Testing

To run all tests:
```bash
uv run pytest
```

To run specific tests:
```bash
uv run pytest pipedrive/api/features/persons/tools/tests/test_person_create_tool.py -v
```

### Package Management

This project uses `uv` for package management:

```bash
uv pip install -e .  # Install package in development mode
```

## Project Structure

```
pipedrive/
├── __init__.py
├── api/
│   ├── __init__.py
│   ├── base_client.py                       (Core HTTP client functionality)
│   ├── pipedrive_api_error.py               (Custom error handling for API responses)
│   ├── pipedrive_client.py                  (Main client that delegates to feature-specific clients)
│   ├── pipedrive_context.py                 (Context manager for MCP integration)
│   ├── features/
│   │   ├── __init__.py                      (Feature discovery mechanism)
│   │   ├── tool_registry.py                 (Feature registry system)
│   │   ├── tool_decorator.py                (Feature-aware tool decorator)
│   │   ├── persons/                         (Person feature module)
│   │   │   ├── __init__.py
│   │   │   ├── persons_tool_registry.py     (Person feature registry)
│   │   │   ├── client/                      (Person-specific API client)
│   │   │   ├── models/                      (Person data models)
│   │   │   └── tools/                       (Person MCP tools)
│   │   ├── organizations/                   (Organization feature module)
│   │   ├── deals/                           (Deal feature module)
│   │   ├── leads/                           (Leads feature module)
│   │   ├── item_search/                     (Item search feature module)
│   │   └── shared/                          (Shared utilities across features)
│   └── tests/                               (Tests for core API components)
├── mcp_instance.py                          (MCP server instance configuration)
└── feature_config.py                        (Feature configuration management)
```

## Project Architecture

### Core Components

1. **Base Client:** (`pipedrive/api/base_client.py`) Provides common HTTP request functionality used by all feature-specific clients.

2. **Pipedrive Client:** (`pipedrive/api/pipedrive_client.py`) Main client that delegates to resource-specific clients like the Person client.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wirasm/pipedrive-mcp](https://github.com/Wirasm/pipedrive-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
