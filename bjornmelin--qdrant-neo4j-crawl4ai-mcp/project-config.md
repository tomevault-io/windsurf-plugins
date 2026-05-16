---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **Agentic RAG MCP Server** that combines vector search (Qdrant), knowledge graphs (Neo4j), and web intelligence (Crawl4AI) through a unified Model Context Protocol (MCP) interface. The system provides autonomous query routing and result fusion using FastMCP 2.0.

## Key Architecture

- **FastMCP 2.0**: Service composition pattern with mounted endpoints
- **Async-First**: All operations use async/await for high concurrency
- **Service Layer**: Three main services (Vector, Graph, Web) with unified interfaces
- **MCP Tools**: Individual tool registrations for each service type
- **Production-Ready**: JWT auth, rate limiting, monitoring, security headers

## Build System & Dependencies

### Primary Tools

- **uv**: Package management (replaces pip) - use `uv sync` for installation
- **ruff**: Code formatting and linting - run `ruff check . --fix` then `ruff format .`
- **mypy**: Type checking - run `mypy .`
- **pytest**: Testing framework with async support

### Development Commands

```bash
# Install dependencies
uv sync

# Install with dev dependencies
uv sync --dev

# Run the server locally
uv run python -m qdrant_neo4j_crawl4ai_mcp

# Run with Docker
docker-compose up -d

# Run tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=qdrant_neo4j_crawl4ai_mcp --cov-report=html

# Run specific test suites
uv run pytest tests/unit/
uv run pytest tests/integration/
uv run pytest tests/security/

# Code quality checks
uv run ruff check . --fix
uv run ruff format .
uv run mypy .

# Security scanning
uv run bandit -r src/
```

### Test Execution

- **Unit tests**: `tests/unit/` - Fast, isolated component tests
- **Integration tests**: `tests/integration/` - Service integration tests  
- **Security tests**: `tests/security/` - Authentication and authorization tests
- **Performance tests**: `tests/performance/` - Load and benchmark tests
- **Property tests**: `tests/property/` - MCP protocol compliance tests

## Architecture Components

### Core Services (src/qdrant_neo4j_crawl4ai_mcp/services/)

1. **VectorService** (`vector_service.py`): Qdrant integration for semantic search
2. **GraphService** (`graph_service.py`): Neo4j integration for knowledge graphs  
3. **WebService** (`web_service.py`): Crawl4AI integration for web intelligence

### MCP Tools (src/qdrant_neo4j_crawl4ai_mcp/tools/)

1. **vector_tools.py**: MCP tool registrations for vector operations
2. **graph_tools.py**: MCP tool registrations for graph operations
3. **web_tools.py**: MCP tool registrations for web operations

### Data Models (src/qdrant_neo4j_crawl4ai_mcp/models/)

- **vector_models.py**: Pydantic models for vector operations
- **graph_models.py**: Pydantic models for graph operations
- **web_models.py**: Pydantic models for web operations

### Core Application Files

- **main.py**: FastAPI application with FastMCP integration and service lifecycle
- **config.py**: Comprehensive settings with Pydantic validation and secrets handling
- **auth.py**: JWT authentication with scope-based authorization
- **middleware.py**: Security middleware stack (CORS, rate limiting, security headers)

## Key Development Patterns

### Service Pattern

Each service follows this structure:

- **Config class**: Pydantic settings with validation
- **Service class**: Business logic with async methods
- **Client integration**: Database/API client management
- **Error handling**: Structured exceptions with proper logging
- **Health checks**: Individual service health endpoints

### MCP Tool Registration

Tools are registered in dedicated modules:

```python
def register_vector_tools(mcp: FastMCP, service: VectorService) -> None:
    @mcp.tool()
    async def vector_search(query: str) -> VectorSearchResponse:
        # Tool implementation
```

### Configuration Management

All configuration uses Pydantic with environment variable support:

```python
class Settings(BaseSettings):
    qdrant_url: str = Field(env="QDRANT_URL", default="http://localhost:6333")
    
    class Config:
        env_file = ".env"
```

## Security Considerations

- **JWT Authentication**: All endpoints require valid JWT tokens with appropriate scopes
- **Rate Limiting**: Redis-backed distributed rate limiting
- **Input Validation**: Comprehensive Pydantic validation on all inputs
- **OWASP Compliance**: Security headers and API security best practices
- **Secrets Management**: SecretStr for sensitive configuration values

## Environment Setup

Required environment variables:

```env
# Database URLs
QDRANT_URL=http://localhost:6333
NEO4J_URI=bolt://localhost:7687
NEO4J_PASSWORD=password
REDIS_URL=redis://localhost:6379/0

# Security
JWT_SECRET_KEY=your-secret-key
ADMIN_API_KEY=admin-key

# External APIs (optional)
OPENAI_API_KEY=sk-...
```

## Service Dependencies

- **Qdrant**: Vector database (port 6333)
- **Neo4j**: Graph database (port 7687)  
- **Redis**: Cache and rate limiting (port 6379)
- **External APIs**: OpenAI (for GraphRAG), various web APIs

## Monitoring & Health Checks

- **Health endpoint**: `/health` - Comprehensive service status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BjornMelin/qdrant-neo4j-crawl4ai-mcp](https://github.com/BjornMelin/qdrant-neo4j-crawl4ai-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
