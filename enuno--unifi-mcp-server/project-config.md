---
trigger: always_on
description: This is a Python-based MCP (Model Context Protocol) server providing access to UniFi Network Controller API.
---

# Cline Rules for UniFi MCP Server

## Project Overview
This is a Python-based MCP (Model Context Protocol) server providing access to UniFi Network Controller API.
- **Tech Stack**: Python 3.10+, FastMCP, asyncio, Pydantic, Redis (optional caching), agnost.ai (optional tracking)
- **Architecture**: Async-first MCP server with 40+ tools, 4 resource endpoints, webhook support
- **Purpose**: Enable AI agents to interact with UniFi network infrastructure via standardized MCP
- **API Status**: Uses UniFi Early Access API (read-only operations currently; write operations available in v1 Stable)

## Core Development Principles

### 1. Documentation and Code Quality
- **ALWAYS** update docstrings when modifying functions
- Follow Google-style docstrings for all public APIs
- Maintain comprehensive inline comments for complex logic
- Update relevant documentation files (README.md, API.md, CONTRIBUTING.md) when adding features
- All code changes MUST pass pre-commit hooks before committing

### 2. Async Programming Standards
- Use `async/await` for ALL I/O-bound operations (API calls, database queries, file operations)
- **NEVER** use blocking synchronous calls in async contexts
- Leverage `asyncio.gather()` for concurrent operations
- Implement proper exception handling with `try/except` blocks in async functions
- Use `async with` for context managers (HTTP clients, database connections)

### 3. Type Safety and Validation
- **ALWAYS** use type hints for function parameters and return values
- Leverage Pydantic models for complex data structures and validation
- Use `Optional[T]` or `T | None` for nullable parameters
- Implement input validation at tool boundaries using Pydantic
- Run `mypy` type checking before submitting PRs

### 4. Testing Requirements
- Target: 80%+ code coverage for all new features
- Write unit tests for individual functions using pytest
- Create integration tests for API interactions (mark with `@pytest.mark.integration`)
- Mock external dependencies (UniFi API, Redis) in unit tests
- Use fixtures for common test setup and teardown
- **NEVER** commit code that breaks existing tests

### 5. Security and Safety
- **CRITICAL**: All mutating operations MUST require `confirm=True` parameter
- Implement dry-run mode (`dry_run=True`) for preview-before-apply functionality
- Log all operations to `audit.log` with timestamps and user context
- Mask sensitive data (passwords, API keys) in logs using utility functions
- Validate all user inputs at API boundaries
- Follow principle of least privilege for API access

## Project Structure Conventions

### Directory Organization
```
src/
├── main.py              # MCP server entry point (registers 40+ tools)
├── config/              # Configuration management (Pydantic Settings)
├── api/                 # UniFi API client (rate limiting, retries, auth)
├── models/              # Pydantic models for data structures
│   ├── device.py        # Device models
│   ├── client.py        # Client models
│   ├── network.py       # Network models
│   ├── site.py          # Site models
│   └── ...              # Additional domain models
├── tools/               # MCP tool implementations (40+ tools)
│   ├── clients.py       # Client query tools
│   ├── devices.py       # Device management
│   ├── networks.py      # Network configuration
│   ├── firewall.py      # Firewall rules
│   ├── wifi.py          # WiFi/SSID management
│   ├── dpi.py           # DPI statistics
│   └── ...              # Additional tool modules
├── resources/           # MCP resource endpoints (4 resources)
├── webhooks/            # Webhook handlers (HMAC verification)
├── utils/               # Utility functions, validators, exceptions
│   ├── exceptions.py    # Custom exception classes
│   ├── validators.py    # Input validation helpers
│   ├── audit.py         # Audit logging
│   └── logger.py        # Logging utilities
└── cache.py             # Redis caching implementation
tests/
├── unit/                # Unit tests (fast, no external deps)
└── integration/         # Integration tests (require UniFi controller)
```

### File Naming
- Use snake_case for Python files: `device_control.py`, `network_config.py`
- Group related tools in single files (e.g., all device tools in `devices.py`)
- Test files mirror source structure: `tests/unit/tools/test_devices.py`

## Coding Standards

### Function Design
- Keep functions focused on single responsibility
- Prefer small, composable functions over large monoliths
- Use descriptive function names: `create_firewall_rule()` not `create()`
- Limit function parameters (max 5-7); use Pydantic models for complex inputs
- Return explicit types; avoid returning `Any` or untyped dicts

### Error Handling
```python
from src.utils.exceptions import (
    APIError,
    AuthenticationError,
    RateLimitError,
    ResourceNotFoundError,
    ValidationError,
    NetworkError,
    ConfirmationRequiredError
)

# GOOD: Specific exception handling with context
try:
    result = await unifi_api.get_devices(site_id)
except RateLimitError as e:
    logger.warning(f"Rate limit exceeded, retry after {e.retry_after}s")
    raise  # Re-raise for MCP protocol handling
except AuthenticationError as e:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enuno/unifi-mcp-server](https://github.com/enuno/unifi-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
