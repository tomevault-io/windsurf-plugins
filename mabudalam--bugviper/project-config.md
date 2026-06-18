---
trigger: always_on
description: This document provides essential information for AI agents working in the BugViper repository.
---

# AGENTS.md - BugViper Codebase Guide

This document provides essential information for AI agents working in the BugViper repository.

## Project Overview

BugViper is an AI-powered code review and repository intelligence platform. It ingests repositories into a Neo4j knowledge graph via Tree-sitter AST parsing and provides:
- Full-text and semantic code search
- LangGraph-powered PR review agent
- AI chat interface for codebase queries

**Tech Stack:**
- Backend: Python 3.13+, FastAPI, Neo4j, LangGraph
- Frontend: Next.js 16, React 19, TypeScript, TailwindCSS
- Package Manager: `uv` (Python), `npm` (frontend)

---

## Build, Lint, and Test Commands

### Python Backend

**Install dependencies:**
```bash
uv sync
```

**Format code:**
```bash
black .
```

**Lint:**
```bash
ruff check .
```

**Type check:**
```bash
mypy .
```

**Run tests:**
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov

# Run a specific test file
pytest tests/test_module.py

# Run a specific test function
pytest tests/test_module.py::test_function_name

# Run tests with verbose output
pytest -v
```

**Run development server:**
```bash
# API server only
uvicorn api.app:app --host 0.0.0.0 --port 8000 --reload

# Run all services (API, Ingestion, Lint, Frontend, Ngrok)
./start.sh
```

**Database setup:**
```bash
# Initialize Neo4j schema and indexes
curl -X POST http://localhost:8000/api/v1/ingest/setup
```

### Frontend (TypeScript/Next.js)

**Install dependencies:**
```bash
cd apps/frontend && npm install
```

**Lint:**
```bash
cd apps/frontend && npm run lint
```

**Build:**
```bash
cd apps/frontend && npm run build
```

**Development server:**
```bash
cd apps/frontend && npm run dev
# Runs on http://localhost:3000
```

---

## Code Style Guidelines

### Python

**Formatting:**
- Line length: 100 characters (configured in pyproject.toml)
- Use `black` for automatic formatting
- Use `ruff` for linting

**Imports:**
- Order: standard library → third-party → local imports
- Use absolute imports for project modules
- Example:
  ```python
  import logging
  import os
  from typing import Any, Dict, List, Optional
  
  from fastapi import APIRouter, Depends, HTTPException
  from neo4j import GraphDatabase
  
  from api.dependencies import get_neo4j_client
  from db.client import Neo4jClient
  ```

**Type hints:**
- Required for all function parameters and return values
- Use `Optional[T]` for optional parameters
- Use `list[T]`, `dict[str, Any]` (lowercase) for Python 3.13+
- Example:
  ```python
  def run_query(
      self, 
      query: str, 
      parameters: Optional[Dict[str, Any]] = None, 
      max_retries: int = 3
  ) -> Tuple[List[Any], Any, List[str]]:
  ```

**Naming conventions:**
- Functions/variables: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Private methods: `_leading_underscore`

**Docstrings:**
- Use triple-quoted docstrings with Args/Returns sections
- Example:
  ```python
  def search_code(query: str, limit: int = 30) -> Dict[str, Any]:
      """
      Search code in the Neo4j graph.
      
      Args:
          query: Search term or keyword
          limit: Maximum results to return
      
      Returns:
          Dictionary with 'results', 'total', and 'query' keys
      """
  ```

**Error handling:**
- Use specific exception types
- In FastAPI routes: raise `HTTPException(status_code=..., detail=...)`
- Log errors before re-raising
- Example:
  ```python
  try:
      results = query_service.search_code(query, repo_id=repo_id)
  except Exception as e:
      logger.error("Search failed: %s", e)
      raise HTTPException(status_code=500, detail=f"Search failed: {str(e)}")
  ```

**Logging:**
- Use `logging.getLogger(__name__)` pattern
- Example:
  ```python
  logger = logging.getLogger(__name__)
  logger.info("Connected to Neo4j database")
  logger.warning("Connection failed: %s", e)
  logger.error("Search failed: %s", str(e))
  ```

**API routes:**
- Use FastAPI's dependency injection for database connections
- Example:
  ```python
  router = APIRouter()
  
  def get_query_service(db: Neo4jClient = Depends(get_neo4j_client)) -> CodeSearchService:
      return CodeSearchService(db)
  
  @router.get("/search")
  async def search_code(
      query: str = Query(..., description="Search term"),
      limit: int = Query(30, description="Maximum results"),
      query_service: CodeSearchService = Depends(get_query_service),
  ) -> Dict[str, Any]:
  ```

### TypeScript/React

**Formatting:**
- Use ESLint with eslint-config-next
- No explicit `any` types (warned)
- Prefer functional components with arrow functions

**Imports:**
- Use path alias `@/` for imports from frontend root
- Use `import type` for type-only imports
- Example:
  ```typescript
  import type { Metadata } from "next";
  import { Toaster } from "@/components/ui/sonner";
  import { AuthProvider } from "@/lib/auth-context";
  ```

**Type definitions:**
- Define interfaces for all API responses
- Use strict null checking (strict mode enabled)
- Example:
  ```typescript
  export interface RepositoryStatistics {
    files: number;
    classes: number;
    functions: number;
    methods: number;
    lines: number;
    imports: number;
    languages: string[];
  }
  ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MabudAlam/BugViper](https://github.com/MabudAlam/BugViper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
