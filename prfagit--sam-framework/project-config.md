---
trigger: always_on
description: SAM (Solana Agent Middleware) is a production-ready AI agent framework for Solana blockchain operations. It provides 15+ tools for automated trading, portfolio management, market data analysis, and web intelligence.
---

# SAM Framework - Cursor Rules
# AI-powered agent framework for Solana blockchain operations

## Project Overview
SAM (Solana Agent Middleware) is a production-ready AI agent framework for Solana blockchain operations. It provides 15+ tools for automated trading, portfolio management, market data analysis, and web intelligence.

## Core Architecture Principles

### Event-Driven Design
- Use async pub/sub messaging system (`sam/core/events.py`)
- All major operations should emit events for observability
- Event names follow pattern: `component.action` (e.g., `llm.usage`, `tool.executed`)

### Plugin Architecture
- Tools registered via Python entry points
- Follow plugin registration pattern in `examples/plugins/`
- Use `ToolSpec` with JSON schema validation
- Include namespace and version for tool organization

### Security-First Approach
- NEVER commit secrets or private keys
- Use Fernet encryption for sensitive data storage
- Implement OS keyring integration for credentials
- Validate all inputs with dedicated validators
- Rate limit all external API calls

## Development Standards

### Python Requirements
- **Python 3.11+** minimum
- **Async-first** architecture using `asyncio` and `uvloop`
- **Type safety** enforced with mypy and Pydantic models
- **100-character line length** (ruff configured)

### Code Organization
```
sam/
├── cli.py                 # Main CLI entry point
├── core/                  # Agent orchestration, LLM, memory, tools
├── integrations/          # Solana, Pump.fun, Jupiter, DexScreener
├── config/               # Settings, prompts, configuration
├── utils/                # Security, validation, utilities
└── commands/             # CLI subcommands
```

### Naming Conventions
- **snake_case**: functions, variables, methods
- **PascalCase**: classes, exceptions
- **SCREAMING_SNAKE_CASE**: constants, environment variables
- **Tool names**: descriptive, action-oriented (e.g., `get_balance`, `pump_fun_buy`)

### Import Organization
```python
# Grouped by: stdlib → third-party → local
import os
import asyncio
from typing import Dict, Any

import aiohttp
from pydantic import BaseModel

from sam.core.tools import Tool, ToolSpec
from sam.utils.validators import validate_address
```

## Tool Development Guidelines

### Tool Registration Pattern
```python
from sam.core.tools import Tool, ToolSpec
from pydantic import BaseModel

class ToolInput(BaseModel):
    parameter: str = Field(..., description="Parameter description")

async def tool_handler(args: Dict[str, Any]) -> Dict[str, Any]:
    """Tool implementation with error handling."""
    return {"success": True, "data": result}

def register(registry, agent=None):
    registry.register(Tool(
        spec=ToolSpec(
            name="tool_name",
            description="What this tool does",
            input_schema={"type": "object", "properties": {...}},
            namespace="integration_name",
            version="1.0.0"
        ),
        handler=tool_handler,
        input_model=ToolInput
    ))
```

### Error Handling Standards
```python
from sam.utils.error_handling import handle_error_gracefully
from sam.utils.error_messages import get_error_message

@handle_error_gracefully
async def tool_function(args: Dict[str, Any]) -> Dict[str, Any]:
    try:
        # Implementation
        return {"success": True, "result": data}
    except Exception as e:
        return {
            "success": False,
            "error": get_error_message("operation_failed"),
            "error_detail": {"code": "specific_error", "message": str(e)}
        }
```

### Validation Requirements
- Use Pydantic models for input validation
- Implement address validation for Solana operations
- Validate transaction amounts against safety limits
- Check slippage parameters (1-50% range)

## CLI Development

### Command Structure
```python
# Add to sam/cli.py main() function
if args.command == "new_command":
    result = await handle_new_command(args)
    return result
```

### Interactive Features
- Use `inquirer` for menu selections when available
- Provide fallback for non-interactive environments
- Include helpful command examples in help text
- Support ESC key interruption for long-running tasks

### Status Display
- Show real-time context usage percentage
- Display wallet address (truncated for security)
- Include session statistics and performance metrics
- Use subtle ANSI colors for better UX

## Testing Standards

### Test Organization
```python
# tests/test_feature.py
import pytest
import pytest_asyncio
from sam.core.feature import FeatureClass

@pytest.mark.asyncio
async def test_feature_functionality():
    # Test implementation
    pass

@pytest.fixture
async def mock_external_api():
    # Mock external dependencies
    pass
```

### Test Categories
- **Unit tests**: Individual functions and classes
- **Integration tests**: Tool interactions and API calls
- **End-to-end tests**: Complete agent workflows
- **Security tests**: Input validation and error handling

### Coverage Requirements
- Minimum 80% code coverage
- Include edge cases and error conditions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prfagit/sam-framework](https://github.com/prfagit/sam-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
