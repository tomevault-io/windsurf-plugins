---
trigger: always_on
description: This document provides AI coding assistants with specific guidance for working with the Insights MCP codebase. **Start by reading [README.md](README.md)** for project overview, authentication setup, and client integrations.
---

# Insights MCP - AI Coding Assistant Guide

This document provides AI coding assistants with specific guidance for working with the Insights MCP codebase. **Start by reading [README.md](README.md)** for project overview, authentication setup, and client integrations.

This guide supplements the README with development-specific information and workflow guidance for AI coding assistants.

## Architecture

### Core Components

1. **InsightsMCPServer** (`src/insights_mcp/server.py`)
   - Main unified server that mounts multiple service toolsets
   - Handles authentication and configuration for all mounted toolsets
   - Supports selective toolset registration via `--toolset` argument

2. **InsightsMCP Base Class** (`src/insights_mcp/mcp.py`)
   - Abstract base class for all Insights MCP toolsets
   - Manages authentication and client initialization
   - Provides common functionality for Red Hat Insights API integration

3. **InsightsClient** (`src/insights_mcp/client.py`)
   - HTTP client for Red Hat Insights APIs with OAuth2 support
   - Handles service account and refresh token authentication
   - Provides error handling and proxy support

4. **OAuth Middleware** (`src/insights_mcp/oauth.py`)
   - Starlette middleware for OAuth flows in HTTP/SSE transports
   - Dynamic client registration and metadata proxying

### Toolset Architecture

The project uses a **toolset-based architecture** where each service is implemented as a separate toolset:

**Toolset Implementation Pattern:**
- Most toolsets: `src/<toolset_name>_mcp/server.py` (e.g., `src/vulnerability_mcp/server.py`)
- Legacy pattern: `src/insights_mcp/servers/<name>.py` (e.g., example toolset)

**Example toolsets (see `src/insights_mcp/server.py` MCPS list for complete current list):**
- **image-builder** (`src/image_builder_mcp/server.py`): Linux image building tools
- **vulnerability** (`src/vulnerability_mcp/server.py`): Security vulnerability management
- **inventory** (`src/inventory_mcp/server.py`): System inventory management

**Toolset Registration:**
- Toolsets are registered in `MCPS` list in `src/insights_mcp/server.py`
- Each toolset has a unique `toolset_name` for identification
- Tools are mounted with toolset prefix (e.g., `image-builder_get_blueprints`)
- Users can select specific toolsets: `insights-mcp --toolset=image-builder`

### Transport Modes

- **STDIO** (default): Direct process communication for desktop integrations
- **HTTP**: RESTful API with streaming capabilities
- **SSE**: Server-sent events for real-time web clients

## Development Workflow for AI Assistants

### Quick Setup
```bash
# Prerequisites: Python 3.10+, uv package manager
uv venv && source .venv/bin/activate
make install-test-deps  # Installs all dev dependencies
```

### Essential Commands
```bash
make help  # Show all available make targets
```

**Key development commands:**
- `make test` - Run test suite
- `make lint` - Run all linting
- `make build` - Build container image
- `insights-mcp` - Run server in development mode

**Note**: Commands require `source .venv/bin/activate` first.
**Note**: Authentication setup is covered in [README.md](README.md).

## Testing

### Test Structure

**General pattern for toolset testing:**
- `tests/` - Main test directory with cross-toolset auth and utility tests
- `src/<toolset_name>_mcp/tests/` - Toolset-specific tests (when present)
- `src/<toolset_name>_mcp/test_prompts.md` - Test prompts for LLM validation

**Example test implementations:**
- `tests/` - Cross-toolset authentication, API, and pattern tests
- `src/image_builder_mcp/tests/` - Full test suite with unit and LLM integration tests
- `src/vulnerability_mcp/test_prompts.md` - LLM test prompts (pattern used by most toolsets)

### Running Tests

**Available test targets:**
```bash
make help  # Shows all available targets with descriptions
```

**Key test commands (see `make help` for complete list):**
- `make test` - Standard test run
- `make test-verbose` - With logging output
- `make test-coverage` - With coverage reporting
- `make install-test-deps` - Install test dependencies

**Manual pytest execution:**
```bash
env DEEPEVAL_TELEMETRY_OPT_OUT=YES uv run pytest -v
```

see also [usage.md](usage.md) for more details on the CLI.

### Test Configuration

1. **Copy example configuration:**
   ```bash
   cp test_config.json.example test_config.json
   ```

2. **Configure LLM models** in `test_config.json`:
   ```json
   {
     "llm_configurations": [{
       "name": "Primary Model",
       "MODEL_ID": "granite-3.1",
       "MODEL_API": "https://your-vLLM-server",
       "USER_KEY": "your-api-key"
     }],
     "guardian_llm": {
       "name": "Evaluation Model",
       "MODEL_ID": "granite-3.2",
       "MODEL_API": "https://your-vLLM-server2",
       "USER_KEY": "your-api-key"
     }
   }
   ```

### Test Types

- **Unit Tests**: Component isolation and API method validation
- **Integration Tests**: End-to-end workflow testing
- **LLM Behavioral Tests**: Validates AI assistant interaction patterns
- **Multi-Transport Tests**: Validates across stdio/HTTP/SSE modes

## Code Quality & Linting


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedHatInsights/insights-mcp](https://github.com/RedHatInsights/insights-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
