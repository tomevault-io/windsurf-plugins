---
trigger: always_on
description: This project uses Redis 8, which is the redis:8 docker image.
---

# CLAUDE.md - Redis Agent Memory Server Project Context

## Redis Version
This project uses Redis 8, which is the redis:8 docker image.
Do not use Redis Stack or other earlier versions of Redis.

## Frequently Used Commands

### Project Setup
Get started in a new environment by installing `uv`:
```bash
pip install uv               # Install uv (once)
make setup                   # Create .venv, sync deps, install pre-commit hooks
make sync                    # Sync latest dependencies
```

### Activate the virtual environment
You MUST always activate the virtualenv before running commands:

```bash
source .venv/bin/activate
```

### Running Tests
Always run tests before committing. You MUST have 100% of the tests in the
code basepassing to commit.

Run all tests like this, including tests that require API keys in the
environment:
```bash
make test-api
```

### Linting

```bash
make pre-commit              # Run the exact formatting/lint hooks used in CI
make verify                  # Run full local verification (pre-commit + API tests)
```

### Managing Dependencies
uv add <dependency>          # Add a dependency to pyproject.toml and update lock file
uv remove <dependency>       # Remove a dependency from pyproject.toml and update lock file

### Running Servers
# Server commands
uv run agent-memory api      # Start REST API server (default port 8000)
uv run agent-memory mcp      # Start MCP server (stdio mode)
uv run agent-memory mcp --mode sse --port 9000  # Start MCP server (SSE mode)

### Database Operations
# Database/Redis operations
uv run agent-memory rebuild-index     # Rebuild Redis search index
uv run agent-memory migrate-memories  # Run memory migrations

### Background Tasks
# Background task management
uv run agent-memory task-worker       # Start background task worker
# Schedule a specific task
uv run agent-memory schedule-task "agent_memory_server.long_term_memory.compact_long_term_memories"

### Running All Containers
# Docker development
docker-compose up            # Start full stack (API, MCP, Redis)
docker-compose up redis      # Start only Redis Stack
docker-compose down          # Stop all services
```

### Committing Changes
IMPORTANT: This project uses `pre-commit`. You should run `pre-commit`
before committing:
```bash
make pre-commit
```

## Important Architectural Patterns

### Dual Interface Design (REST + MCP)
- **REST API**: Traditional HTTP endpoints for web applications (`api.py`)
- **MCP Server**: Model Context Protocol for AI agent integration (`mcp.py`)
- Both interfaces share the same core memory management logic

### Memory Architecture
```python
# Two-tier memory system
Working Memory (Session-scoped)  →  Long-term Memory (Persistent)
    ↓                                      ↓
- Messages                          - Semantic search
- Context                          - Topic modeling
- Structured memories              - Entity recognition
- Metadata                         - Deduplication
```

### RedisVL Integration
**CRITICAL**: Always use RedisVL query types instead of direct redis-py client access for searches:
```python
# Correct - Use RedisVL queries
from redisvl.query import VectorQuery, FilterQuery
query = VectorQuery(vector=embedding, vector_field_name="vector", return_fields=["text"])

# Avoid - Direct redis client searches
# redis.ft().search(...)  # Don't do this
```

### Async-First Design
- All core operations are async
- Background task processing with Docket
- Async Redis connections throughout

## Critical Rules

### Import Placement
Place all imports at the top of modules, not inside functions. Inline imports should only be used when strictly necessary (e.g., avoiding circular dependencies, optional dependencies, or significant startup performance concerns).

### Authentication
- **PRODUCTION**: Never set `DISABLE_AUTH=true` in production
- **DEVELOPMENT**: Use `DISABLE_AUTH=true` for local testing only
- JWT/OAuth2 authentication required for all endpoints except `/health`, `/docs`, `/openapi.json`

### Memory Management
- Working memory automatically promotes structured memories to long-term storage
- Conversations are summarized when exceeding window size
- Use model-aware token limits for context window management

### RedisVL Usage (Required)
Always use RedisVL query types for any search operations. This is a project requirement.

## Testing Notes

The project uses `pytest` with `testcontainers` for Redis integration testing:

- `make test` - Run the standard test suite
- `make test-api` - Run all tests including API-key-dependent tests
- `make test-unit` - Unit tests only
- `make test-integration` - Integration tests (require Redis)
- `make test-cov` - Run tests with coverage

## Project Structure

```
agent_memory_server/
├── main.py              # FastAPI application entry point
├── api.py               # REST API endpoints
├── mcp.py               # MCP server implementation
├── config.py            # Configuration management
├── auth.py              # OAuth2/JWT authentication
├── models.py            # Pydantic data models
├── working_memory.py    # Session-scoped memory management
├── long_term_memory.py  # Persistent memory with semantic search
├── messages.py          # Message handling and formatting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redis/agent-memory-server](https://github.com/redis/agent-memory-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
