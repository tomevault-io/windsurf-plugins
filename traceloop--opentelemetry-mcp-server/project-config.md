---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Opentelemetry MCP Server (`opentelemetry-mcp`) is an MCP (Model Context Protocol) server that enables AI agents to query and analyze OpenTelemetry traces from LLM applications. It parses Opentelemetry semantic conventions (the `gen_ai.*` attributes) to enable automated debugging and observability.

**Key Features:**

- Multi-backend support: Jaeger, Grafana Tempo, and Traceloop
- 9 MCP tools: Core tools + LLM-oriented discovery and analysis tools
- Token usage tracking and aggregation across models/services
- Finish reasons tracking for debugging truncated/filtered responses
- Enhanced token calculation supporting all `gen_ai.usage.*` attributes
- Dual transport modes: stdio (Claude Desktop) and HTTP/SSE (network access)

## Development Commands

**Package Manager:** This project uses UV (not pip). All commands should use `uv run`.

```bash
# Install dependencies
uv sync

# Run the server (stdio transport for Claude Desktop)
uv run opentelemetry-mcp

# Run with HTTP transport
uv run opentelemetry-mcp --transport http --port 8000

# Override backend configuration
uv run opentelemetry-mcp --backend jaeger --url http://localhost:16686

# Run all tests
uv run pytest

# Run with coverage
uv run pytest --cov=opentelemetry_mcp --cov-report=html

# Run specific test file
uv run pytest tests/test_models.py

# Format code (always run before committing)
uv run ruff format .

# Lint code
uv run ruff check .

# Type check (strict mode, must pass)
uv run mypy src/
```

## Architecture

### Backend Abstraction Pattern

All trace storage backends implement the `BaseBackend` abstract interface in [opentelemetry_mcp/backends/base.py](opentelemetry_mcp/backends/base.py):

```python
class BaseBackend(ABC):
    @abstractmethod
    async def search_traces(self, query: TraceQuery) -> list[TraceData]: ...
    @abstractmethod
    async def search_spans(self, query: SpanQuery) -> list[SpanData]: ...
    @abstractmethod
    async def get_trace(self, trace_id: str) -> TraceData: ...
    @abstractmethod
    async def list_services(self) -> list[str]: ...
    @abstractmethod
    async def get_service_operations(self, service: str) -> list[str]: ...
    @abstractmethod
    async def health_check(self) -> bool: ...
```

Concrete implementations:

- [backends/jaeger.py](opentelemetry_mcp/backends/jaeger.py) - Jaeger backend
- [backends/tempo.py](opentelemetry_mcp/backends/tempo.py) - Grafana Tempo backend
- [backends/traceloop.py](opentelemetry_mcp/backends/traceloop.py) - Traceloop backend

### Tool-Based Architecture

Each MCP capability is implemented as a separate tool module in [opentelemetry_mcp/tools/](opentelemetry_mcp/tools/):

**Core Tools:**

- [tools/search.py](opentelemetry_mcp/tools/search.py) - Search traces with filters
- [tools/search_spans.py](opentelemetry_mcp/tools/search_spans.py) - Search individual spans with filters
- [tools/trace.py](opentelemetry_mcp/tools/trace.py) - Get detailed trace by ID
- [tools/usage.py](opentelemetry_mcp/tools/usage.py) - Aggregate token usage metrics
- [tools/services.py](opentelemetry_mcp/tools/services.py) - List available services
- [tools/errors.py](opentelemetry_mcp/tools/errors.py) - Find traces with errors

**LLM-Oriented Tools (Discovery & Analysis):**

- [tools/list_models.py](opentelemetry_mcp/tools/list_models.py) - List all models in use with statistics
- [tools/model_stats.py](opentelemetry_mcp/tools/model_stats.py) - Performance stats for a specific model
- [tools/expensive_traces.py](opentelemetry_mcp/tools/expensive_traces.py) - Find highest token usage traces
- [tools/slow_traces.py](opentelemetry_mcp/tools/slow_traces.py) - Find slowest LLM traces
- [tools/list_llm_tools.py](opentelemetry_mcp/tools/list_llm_tools.py) - List LLM tools used (via traceloop.span.kind == tool)

**Critical:** All tools MUST return JSON strings (not dicts). This is required by the MCP protocol.

```python
# Correct
return json.dumps({"result": data})

# Incorrect - will break MCP protocol
return {"result": data}
```

### Key Components

- [server.py](opentelemetry_mcp/server.py) - FastMCP application, CLI interface, tool handlers
- [config.py](opentelemetry_mcp/config.py) - Pydantic configuration models
- [models.py](opentelemetry_mcp/models.py) - Core data models (SpanData, TraceData, UsageMetrics)
- [attributes.py](opentelemetry_mcp/attributes.py) - Strongly-typed OpenTelemetry attribute models

## Configuration

**Environment Variables** (see [.env.example](.env.example)):

- `BACKEND_TYPE` - Required: `jaeger`, `tempo`, or `traceloop`
- `BACKEND_URL` - Required: Backend API endpoint
- `BACKEND_API_KEY` - Optional: Authentication key
- `BACKEND_TIMEOUT` - Optional: Request timeout (default: 30s)
- `LOG_LEVEL` - Optional: Logging level (default: INFO)
- `MAX_TRACES_PER_QUERY` - Optional: Result limit (default: 100)

**Configuration Precedence:** CLI args > environment variables > defaults

## Opentelemetry Semantic Conventions

The server parses both current and legacy Opentelemetry conventions:

**Primary (gen_ai.\*):**

- `gen_ai.system` - LLM provider (e.g., "openai", "anthropic")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [traceloop/opentelemetry-mcp-server](https://github.com/traceloop/opentelemetry-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
