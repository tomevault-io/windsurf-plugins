---
trigger: always_on
description: This guide helps AI agents understand the codebase structure and be immediately productive.
---

# Agent Customizations for Reversecore_MCP

This guide helps AI agents understand the codebase structure and be immediately productive.

## Project Overview

**Reversecore_MCP** is an enterprise-grade MCP (Model Context Protocol) server for AI-powered reverse engineering. It enables AI agents to perform comprehensive binary analysis through natural language commands using Ghidra, Radare2, and other industry-standard tools.

- **Language**: Python 3.10+
- **Framework**: [FastMCP](https://github.com/jlowin/fastmcp) 2.13.1+
- **Architecture**: Layered (Prompts → Tools → Core Infrastructure → External Tools)
- **Test Coverage**: 55%+ with 700+ tests
- **Key External Dependencies**: Ghidra (for decompilation), Radare2 (disassembly), YARA (detection)

## Quick Commands

### Development Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/macOS
venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Install pre-commit hooks (enforces code standards)
pre-commit install
```

### Building & Running

```bash
# Start MCP server (development mode)
python server.py

# Docker (auto-detects architecture: Intel/ARM)
docker compose --profile x86 up -d    # Intel/AMD
docker compose --profile arm64 up -d  # Apple Silicon
./scripts/run-docker.sh                # Auto-detect

# Install external tools
./scripts/install-ghidra.sh            # Linux/macOS
.\scripts\install-ghidra.ps1           # Windows
```

### Testing

```bash
# All tests (generates coverage report)
pytest tests/ -v

# Unit tests only
pytest tests/unit/ -v

# Integration tests
pytest tests/integration/ -v

# Specific test with coverage
pytest tests/unit/test_cli_tools.py::TestRunFile::test_success -v

# Check coverage threshold (must exceed 54%)
pytest --cov-report=term-missing
```

### Code Quality

```bash
# Lint with Ruff
ruff check reversecore_mcp/

# Format with Black
black reversecore_mcp/

# Sort imports with isort
isort reversecore_mcp/

# Auto-fix all issues
ruff check --fix reversecore_mcp/
black reversecore_mcp/
```

## Architecture

### High-Level Design

```
AI Agent → MCP Protocol → Reversecore Server (FastMCP)
                              ↓
                    [Prompts] [Resources] [Tools]
                              ↓
                    Core Infrastructure Layer
                (Config, Security, Ghidra, Radare2, Metrics)
                              ↓
              Ghidra  │  Radare2  │  CLI Tools
            (Java)   │ (Python)  │ (YARA, file, etc.)
```

### Key Directories

- **`reversecore_mcp/core/`**: Infrastructure (config, exceptions, security, validation, tool management)
- **`reversecore_mcp/tools/`**: Tool implementations organized by category
  - `analysis/`: Binary analysis (signatures, LIEF, diff, static analysis)
  - `ghidra/`: Decompilation via Ghidra
  - `radare2/`: Disassembly & emulation via Radare2
  - `malware/`: Threat detection (vaccine generation, dormant detector, vulnerability hunter)
  - `report/`: Report generation
  - `common/`: File operations, patch explanation
- **`reversecore_mcp/prompts.py`**: AI reasoning prompts for different analysis tasks
- **`reversecore_mcp/resources.py`**: Dynamic resources exposed to AI agents
- **`server.py`**: FastMCP server entry point

See [Architecture Documentation](docs/development/architecture.md) for details.

## Code Patterns & Conventions

### Tool Implementation Pattern

All MCP tools follow this structure:

```python
@server.tool()
async def my_analysis_tool(file_path: str, options: str | None = None) -> ToolResult:
    """Brief description of what this tool does.

    Args:
        file_path: Path to the binary file.
        options: Optional analysis options.

    Returns:
        ToolResult with status='success'/'error' and appropriate content/error fields.
    """
    try:
        # Validate inputs
        validated_path = validate_file_path(file_path)

        # Perform analysis
        result = await perform_analysis(validated_path)

        # Return success
        return ToolResult(
            status="success",
            content=[{"type": "text", "text": json.dumps(result, indent=2)}]
        )
    except ValidationError as e:
        return ToolResult(status="error", error=str(e))
    except ExecutionTimeoutError as e:
        logger.error(f"Timeout: {e}")
        return ToolResult(status="error", error=str(e))
```

### Exception Handling

All custom exceptions inherit from `ReversecoreError`:

```python
from reversecore_mcp.core.exceptions import ReversecoreError, ValidationError, ExecutionTimeoutError

# Exceptions have error codes (RCMCP-E*) and error types for categorization
class ValidationError(ReversecoreError):
    error_code = "RCMCP-E001"
    error_type = "VALIDATION_ERROR"
```

### Input Validation

Always validate file paths and inputs:

```python
from reversecore_mcp.core.validators import validate_file_path, validate_binary_path

# Validates path exists, is readable, and within workspace
safe_path = validate_file_path(user_input)

# Validates it's actually a binary file
binary_path = validate_binary_path(user_input)
```

### Result Models


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sjkim1127/Reversecore_MCP](https://github.com/sjkim1127/Reversecore_MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
