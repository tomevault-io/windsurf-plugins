---
trigger: always_on
description: **SayAThing** is an open-source Text-to-Speech (TTS) API platform powered by the Kokoro engine and FastAPI. The project provides a robust, scalable TTS service with a worker queue system for handling audio generation tasks.
---

# GitHub Copilot Instructions for SayAThing

## Project Overview

**SayAThing** is an open-source Text-to-Speech (TTS) API platform powered by the Kokoro engine and FastAPI. The project provides a robust, scalable TTS service with a worker queue system for handling audio generation tasks.

### Key Features
- **TTS API**: RESTful API for text-to-speech conversion using Kokoro engine
- **Worker Queue System**: SQLite-backed queue with retry mechanisms for reliable processing
- **Dependency Injection**: Singleton pattern for shared resources using `python-dependency-injector`
- **Voice Management**: Support for multiple voices with preloading capabilities
- **Dashboard**: Web interface for monitoring and management

## Documentation Reference

**Important**: For detailed information about specific topics, always check the documentation in `./docs/` first:

- **`./docs/onboarding.md`**: Complete engineer onboarding guide with codebase overview, development setup, coding standards, and testing practices
- **`./docs/dependency_injection.md`**: Detailed explanation of the DI implementation, singleton patterns, and how to use the container system
- **`./docs/testing.md`**: Comprehensive testing guide including test structure, running tests, fixtures, and integration test configuration
- **`./docs/worker_queue.solution.md`**: In-depth worker queue implementation details, database schema, and operational patterns

When answering questions about:
- **Project setup, development environment, or coding standards** → Reference `./docs/onboarding.md`
- **Dependency injection, container usage, or singleton patterns** → Reference `./docs/dependency_injection.md`
- **Testing approaches, fixtures, or test execution** → Reference `./docs/testing.md`
- **Worker queue operations, database schema, or task management** → Reference `./docs/worker_queue.solution.md`

Always suggest that users consult the relevant documentation for comprehensive details beyond what's covered in this instructions file.

## Architecture & Structure

### Core Components

1. **Server Layer** (`server/`)
   - **FastAPI Application**: Main HTTP server with async request handling
   - **Routes**: Modular API endpoints (`/api/tts`, `/tts/voices`, `/healthz`, dashboard)
   - **Config**: Application configuration and dependency injection setup
   - **Exception Handlers**: Centralized error handling for TTS operations

2. **TTS Engine** (`tts/`)
   - **Engine Interface**: Abstract base for TTS implementations
   - **Kokoro Engine**: Primary TTS implementation with voice preloading
   - **Voice Management**: Voice metadata, loading, and sample generation
   - **Request/Response Models**: Pydantic models for API schemas

3. **Worker System** (`worker/`)
   - **Queue**: SQLite-backed persistent queue with atomic operations
   - **Database**: SQLAlchemy ORM with task state management
   - **Workers**: Primary and retry workers for task processing
   - **Task Management**: ULID-based task IDs with state transitions

4. **Dependency Injection** (`container.py`)
   - **Singleton DatabaseManager**: Shared database instance across all components
   - **Factory Pattern**: WorkerQueue instances with injected dependencies
   - **Test Containers**: Isolated DI containers for testing

### Dependencies & Tech Stack

- **Python 3.12+** (required)
- **FastAPI**: Async web framework with OpenAPI documentation
- **SQLAlchemy**: ORM for database operations
- **SQLite**: Default database (with aiosqlite for async)
- **Kokoro**: TTS engine for voice synthesis
- **dependency-injector**: DI container management
- **pytest**: Testing framework with async support
- **uv**: Package manager (preferred) with lock file support

## Development Guidelines

### Code Style & Conventions

- **Type Hints**: Always use full type annotations for functions and variables
- **Async/Await**: Prefer async patterns for I/O operations
- **Naming Conventions**:
  - Files/modules: `snake_case.py`
  - Variables/functions: `snake_case`
  - Classes: `PascalCase`
  - Constants: `UPPER_SNAKE_CASE`
- **Docstrings**: Use comprehensive docstrings for public APIs
- **Error Handling**: Use custom exception classes with descriptive messages

### File Organization Rules

- **Server Routes**: Keep endpoints in `server/routes/` with clear tagging
- **TTS Engines**: Implement `TTSEngineInterface` in `tts/` directory
- **Worker Logic**: Place worker implementations in `worker/workers/`
- **Configuration**: Use environment variables with defaults in config classes
- **Tests**: Co-locate tests with modules using `test_*.py` naming

### API Design Patterns

- **Request/Response Models**: Use Pydantic for validation and serialization
- **Error Responses**: Standardized HTTP status codes with descriptive messages
- **OpenAPI Documentation**: Include examples, descriptions, and response schemas
- **Async Handlers**: All route handlers should be async functions
- **Dependency Injection**: Use container for shared resources in routes

### Database & Queue Patterns

- **ULID Primary Keys**: Use `ulid-py` for sortable, unique task identifiers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kanthorlabs/sayathing](https://github.com/kanthorlabs/sayathing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
