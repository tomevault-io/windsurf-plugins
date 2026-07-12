---
trigger: always_on
description: You are an expert in building production-ready AI agent applications using Python, FastAPI, LangGraph, and LangChain.
---


# 只要 Cursor 打开这个项目，并且 .cursor/rules/project-main-rules.mdc 存在且 alwaysApply: true，Cursor AI 就会默认参考它。

# LangGraph FastAPI AI Agent Development

You are an expert in building production-ready AI agent applications using Python, FastAPI, LangGraph, and LangChain.

This is a **LangGraph FastAPI Agent Project** for building scalable, secure AI agent services with LLM orchestration, observability, and persistence.

## Project Architecture Overview

This is an AI agent application that:

- Uses **LangGraph** for building stateful, multi-step AI agent workflows
- Uses **FastAPI** for high-performance async REST API endpoints
- Integrates **Langfuse** for LLM observability and tracing
- Uses **PostgreSQL** with **pgvector** for long-term memory storage (mem0ai)
- Implements **JWT authentication** with session management
- Provides **rate limiting** with slowapi
- Includes **Prometheus metrics** and **Grafana dashboards** for monitoring
- Uses **structlog** for structured logging with environment-specific formatting
- Implements **retry logic** using tenacity library
- Uses **rich** library for colored, formatted console outputs

## Key Principles

- Write concise, technical responses with accurate Python examples
- Use functional, declarative programming; avoid classes where possible except for services and agents
- Prefer iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (e.g., `is_active`, `has_permission`)
- Use lowercase with underscores for directories and files (e.g., `routers/user_routes.py`)
- Favor named exports for routes and utility functions
- Use the Receive an Object, Return an Object (RORO) pattern
- **All imports must be at the top of the file** - never add imports inside functions or classes

## Python/FastAPI Conventions

- Use `def` for pure functions and `async def` for asynchronous operations
- Use type hints for all function signatures; prefer Pydantic models over raw dictionaries
- File structure: exported router, sub-routes, utilities, static content, types (models, schemas)
- Use concise, one-line syntax for simple conditional statements (e.g., `if condition: do_something()`)
- Avoid unnecessary else statements; use the if-return pattern instead

## LangGraph & LangChain Integration

- Use LangGraph `StateGraph` for building AI agent workflows with multiple steps/nodes
- Define clear state schemas using Pydantic models (see `app/schemas/graph.py`)
- Use `CompiledStateGraph` for production workflows
- Implement `AsyncPostgresSaver` for checkpointing and persistence
- Use LangChain's `CallbackHandler` from Langfuse for tracing LLM calls
- Structure agents as classes that manage graph creation and execution (see `app/core/langgraph/graph.py`)
- Use `Command` for controlling graph flow between nodes
- Implement proper streaming responses for long-running agent operations

## Long-Term Memory (mem0ai)

- Use mem0ai's `AsyncMemory` for semantic memory storage
- Configure with pgvector as the vector store backend
- Store memories per user_id for personalized experiences
- Use async methods: `add()`, `get()`, `search()`, `delete()`
- Configure memory collection name via environment variables

## Error Handling and Validation

Prioritize error handling and edge cases:

- Handle errors and edge cases at the beginning of functions
- Use early returns for error conditions to avoid deeply nested if statements
- Place the happy path last in the function for improved readability
- Use guard clauses to handle preconditions and invalid states early
- Implement proper error logging with structured logging
- Use `HTTPException` for expected errors with appropriate status codes
- Use middleware for handling unexpected errors globally

## Logging Standards

Use structlog for all logging with these conventions:

- Log messages must be **lowercase and separated by underscores** (e.g., `"user_login_successful"`)
- **No f-strings in structlog events** - pass all variables as kwargs for proper filtering
- Use `logger.exception()` instead of `logger.error()` to preserve tracebacks
- For warnings with exceptions, use `exc_info=True`: `logger.warning("event_name", exc_info=True)`
- Always bind context to logs: session_id, user_id, request_id, etc.
- Use appropriate log levels: `debug`, `info`, `warning`, `error`, `exception`
- Example: `logger.info("chat_request_received", session_id=session.id, message_count=len(messages))`

## Rich Library for Outputs

- **Always enable rich library** for formatted console outputs
- Use rich for progress bars, tables, panels, and formatted text
- Use rich.console for debugging complex data structures
- Apply rich formatting for evaluation reports and CLI outputs

## Retry Logic

- **Always use tenacity library** for retry logic
- Configure retries with exponential backoff
- Set appropriate stop conditions (max attempts, max time)
- Log retry attempts for observability
- Example: `@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))`

## Caching Strategy

- **Only cache successful responses**, never cache errors
- Use appropriate cache TTL based on data volatility
- Implement cache invalidation strategies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jarry126/fastapi-langgraph-agent-zh](https://github.com/jarry126/fastapi-langgraph-agent-zh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
