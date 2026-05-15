---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Mathematics is a production-ready Model Context Protocol (MCP) server providing comprehensive mathematical computation capabilities. Built with FastMCP v2.0+, it exposes 52 advanced mathematical functions, 158 unit conversions across 15 categories, financial calculations, and statistical operations through 21 specialized MCP tools.

**Key Architecture Principle**: AST-based expression evaluation ensures security by preventing code injection while maintaining full mathematical capability.

## Development Commands

### Testing

```bash
# Run all tests (130 test cases)
python -m pytest tests/ -v

# Run tests with coverage
python -m pytest tests/ --cov=src --cov-report=html

# Run specific test categories
python -m pytest tests/test_calculator.py -v # Core functionality
python -m pytest tests/performance/ -v       # Performance tests
python -m pytest tests/security/ -v          # Security validation
python -m pytest tests/precision/ -v         # Numerical precision
python -m pytest tests/validation/ -v        # Input validation
```

### Code Quality

```bash
# Auto-format code (100-character line limit)
black src/ tests/ --line-length 100

# Lint with Ruff
ruff check src/ tests/ --fix

# Type checking
mypy src/

# Security analysis
bandit -r src/

# Run all pre-commit checks
pre-commit run --all-files
```

### Running the Server

```bash
# Development mode (local testing)
python -m mcp_mathematics

# Using uv (recommended for production)
uvx mcp-mathematics

# Install in editable mode for development
pip install -e .
```

### Building and Distribution

```bash
# Clean previous builds
rm -rf dist/ build/

# Build source and wheel distributions
python -m build

# Verify package integrity
twine check dist/*

# Test installation in clean environment
pip install dist/*.whl
```

## Architecture Overview

### Core Components

**Single-File Architecture**: All functionality is consolidated in `src/mcp_mathematics/calculator.py` (~4500 lines) for maximum portability and minimal dependencies.

#### Security Layer (AST Evaluation Engine)

- **Input Validation**: Regex-based pattern matching blocks forbidden operations (`import`, `exec`, `eval`, `__*__`, etc.)
- **AST Parsing**: Python's Abstract Syntax Tree validates expression structure before evaluation
- **Node Depth Limiting**: Maximum AST depth of 10 prevents deeply nested attacks
- **Operation Whitelisting**: Only approved mathematical operations (arithmetic, functions, comparisons) are allowed
- **Resource Limits**: Expression length (1000 chars), computation timeout (15s), memory limits (512MB)

#### Cache System

- **LRUExpressionCache**: Bounded LRU cache with configurable max size (1000 entries)
- **TTLExpressionCache**: Time-based expiry (300s default) for computation results
- **Thread-Safe**: RLock-based synchronization for concurrent access
- **Memory Management**: Automatic cleanup via threading.Timer (no signal-based timeouts)

#### Session Management

- **SessionVariableManager**: Stateful calculation contexts with variable storage
- **Session Isolation**: Independent namespaces per session_id
- **TTL-Based Cleanup**: Automatic session expiry (3600s default, configurable)
- **Bounded Sessions**: Maximum 100 concurrent sessions with LRU eviction

#### Computation Tracking

- **ComputationMetrics**: Thread-safe performance monitoring
- **CalculationHistory**: Deque-based history with configurable limit (100 entries)
- **Rate Limiting**: Per-client request tracking (1M requests/60s window)

### FastMCP Integration

**Configuration**: `fastmcp.json` defines:

- **Source**: `src/mcp_mathematics/calculator.py` with entrypoint `mcp`
- **Environment**: `uv` with Python >=3.10, dependencies: `mcp>=1.4.1`, `fastmcp>=0.1.0`
- **Deployment**: stdio transport, INFO log level

**Tool Annotations**: All 21 tools use FastMCP's `ToolAnnotations` for behavioral metadata (safe, caches, writes, etc.)

### MCP Tools Structure

21 tools organized by domain with descriptive mathematical names:

**Core Calculation**:

1. `evaluate_mathematical_expression` - Single expression evaluation
2. `evaluate_multiple_mathematical_expressions` - Parallel batch processing
3. `convert_between_measurement_units` - 158 units across 15 categories
4. `convert_units_from_natural_language` - NLP-based unit conversion
5. `compute_statistical_operations` - Statistical analysis (mean, median, std dev, etc.)
6. `perform_matrix_mathematical_operations` - Linear algebra (multiply, determinant, inverse)
7. `perform_number_theory_analysis` - Prime testing, factorization, divisors

**Session Management**:
8\. `create_mathematical_calculation_session` - Initialize stateful context
9\. `evaluate_expression_in_session_context` - Session-aware calculations
10\. `list_mathematical_session_variables` - Variable inspection
11\. `delete_mathematical_calculation_session` - Session cleanup

**System Monitoring**:
12\. `performance_metrics` - Performance stats
13\. `security_status` - Security status
14\. `memory_statistics` - Memory analytics

**Management**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SHSharkar/MCP-Mathematics](https://github.com/SHSharkar/MCP-Mathematics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
