---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Installation and Setup
```bash
# Using uv (recommended)
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv pip install -e .

# Using pip
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -e .
```

## Dependency Rules

- Keep Python dependency specifiers exact where this repository already pins them, including test extras and `build-system`.
- If `pyproject.toml` changes affect resolved packages, verify the result through the lock/install path used by CI.
- Do not introduce floating versions in CI or release automation when exact pins are practical.

### Running the MCP Server
```bash
# Basic run with stdio transport
python src/codealive_mcp_server.py

# With debug mode enabled
python src/codealive_mcp_server.py --debug

# With SSE transport
python src/codealive_mcp_server.py --transport sse --host 0.0.0.0 --port 8000

# With custom API key and base URL
python src/codealive_mcp_server.py --api-key YOUR_KEY --base-url https://custom.url
```

### Docker Usage
```bash
# Build Docker image
docker build -t codealive-mcp .

# Run with Docker
docker run --rm -i -e CODEALIVE_API_KEY=your_key_here codealive-mcp
```

### Testing

#### Quick Smoke Test
After making local changes, quickly verify everything works:
```bash
# Using make (recommended)
make smoke-test

# Or directly
python smoke_test.py

# With valid API key for full testing
CODEALIVE_API_KEY=your_key python smoke_test.py
```

The smoke test:
- ✓ Verifies server starts and connects via stdio
- ✓ Checks all tools are registered correctly
- ✓ Tests each tool responds appropriately
- ✓ Validates parameter handling
- ✓ Runs in ~5 seconds

#### Unit Tests
Run comprehensive unit tests with pytest:
```bash
# Using make
make unit-test

# Or directly
pytest src/tests/ -v

# With coverage
pytest src/tests/ -v --cov=src
```

#### All Tests
Run both smoke tests and unit tests:
```bash
make test
```

## Architecture

This is a Model Context Protocol (MCP) server that provides AI clients with access to CodeAlive's semantic code search and analysis capabilities.

### Core Components

- **`codealive_mcp_server.py`**: Main entry point — bootstraps logging, tracing, registers tools and middleware
- **Eight tools**: `get_data_sources`, `semantic_search`, `grep_search`, `fetch_artifacts`, `get_artifact_relationships`, `chat`, `codebase_search`, `codebase_consultant`
- **`core/client.py`**: `CodeAliveContext` dataclass + `codealive_lifespan` (httpx.AsyncClient lifecycle, `_server_ready` flag)
- **`core/logging.py`**: loguru structured JSON logging + PII masking + OTel context injection
- **`core/observability.py`**: OpenTelemetry TracerProvider setup with OTLP export
- **`middleware/`**: `N8NRemoveParametersMiddleware` (strips n8n extra params) + `ObservabilityMiddleware` (OTel spans per tool call)

### Key Architectural Patterns

1. **FastMCP Framework**: Uses FastMCP 3.x with lifespan context, middleware hooks, and built-in `Client` for testing
2. **HTTP Auth via `get_http_headers`**: FastMCP 3.x strips the `authorization` header by default (to prevent accidental credential forwarding to downstream services). Our `get_api_key_from_context()` in `core/client.py` must use `get_http_headers(include={"authorization"})` to read Bearer tokens from HTTP/streamable-http clients. **Do not remove the `include=` parameter** — without it, all HTTP-transport clients (LibreChat, n8n, etc.) will fail with a misleading STDIO-mode error.
3. **HTTP Client Management**: Single persistent `httpx.AsyncClient` with connection pooling, created in lifespan
3. **Streaming Support**: `chat` and the deprecated `codebase_consultant` alias use SSE streaming (`response.aiter_lines()`) for chat completions
4. **Environment Configuration**: Supports both .env files and command-line arguments with precedence
5. **Error Handling**: Centralized in `utils/errors.py` — all tools use `handle_api_error()` with `method=` prefix
6. **N8N Middleware**: Strips extra parameters (sessionId, action, chatInput, toolCallId) from n8n tool calls before validation
7. **Observability Middleware**: Wraps every `tools/call` in an OTel span with GenAI semantic conventions

### Data Flow

1. AI client connects to MCP server via stdio/HTTP transport
2. Client calls tools (`get_data_sources` → `semantic_search` / `grep_search` → `fetch_artifacts` / `get_artifact_relationships` → `chat` only if synthesis is still needed)
3. Middleware chain runs: N8N cleanup → ObservabilityMiddleware (OTel span + log correlation)
4. Tool translates MCP call to CodeAlive API request (with `X-CodeAlive-*` headers)
5. Response parsed and returned to the AI client — as a `dict` for metadata/discovery tools, as an XML string for `fetch_artifacts`, or as plain text for `chat`

### Environment Variables

- `CODEALIVE_API_KEY`: Required API key for CodeAlive service
- `CODEALIVE_BASE_URL`: API base URL (defaults to https://app.codealive.ai)
- `CODEALIVE_IGNORE_SSL`: Set to disable SSL verification (debug mode)
- `DEBUG_MODE`: Set to `true` to enable DEBUG-level logging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CodeAlive-AI/codealive-mcp](https://github.com/CodeAlive-AI/codealive-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
