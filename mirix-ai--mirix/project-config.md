---
trigger: always_on
description: MIRIX is a Multi-Agent Personal Assistant with an Advanced Memory System. It features a six-agent memory architecture (Core, Episodic, Semantic, Procedural, Resource, Knowledge Vault) with screen activity tracking and privacy-first design.
---

# MIRIX Project Rules for Cursor AI

## Project Overview
MIRIX is a Multi-Agent Personal Assistant with an Advanced Memory System. It features a six-agent memory architecture (Core, Episodic, Semantic, Procedural, Resource, Knowledge Vault) with screen activity tracking and privacy-first design.

## Tech Stack
- **Backend**: Python 3.10+, FastAPI, PostgreSQL, Redis, Kafka
- **Frontend**: React, TypeScript, Vite, TailwindCSS
- **LLM Integration**: Multiple providers (OpenAI, Anthropic, Google, Azure, etc.)
- **Observability**: LangFuse
- **Testing**: pytest
- **Linting**: Ruff, Pyright

## Code Standards

### Python Guidelines
1. **Follow PEP 8** strictly
   - Use 4 spaces for indentation (no tabs)
   - Max line length: 79 characters for code, 72 for docstrings
   - Use snake_case for variables/functions, CamelCase for classes, ALL_CAPS for constants
   - Group imports: standard library, third-party, local application (blank lines between groups)

2. **Type Hints**
   - Always use type hints for function signatures
   - Use `Optional[T]` for nullable types
   - Use `Union` or `|` syntax for multiple types
   - Example: `def create_user(user_id: str, name: Optional[str] = None) -> User:`

3. **Documentation**
   - Use triple-quoted docstrings for all modules, classes, and functions
   - Include parameter descriptions, return types, and examples where helpful
   - Document complex logic with inline comments explaining WHY, not WHAT

4. **Error Handling**
   - Use specific exception types (ValueError, TypeError, HTTPException, etc.)
   - Never use bare `except:` clauses
   - Log errors with appropriate context

5. **Async/Await**
   - Use async/await for I/O operations
   - Prefix async functions with `async def`
   - Always await async calls

### Async Architecture (Critical)
The codebase is fully async-native. Violating these rules will break the server.

1. **All new manager methods in `mirix/services/` MUST be `async def`**
2. **Never call `asyncio.run()` inside the server** — the event loop is already running
3. **Never use sync DB drivers** — use `asyncpg` (not `pg8000`), `aiosqlite` (not `sqlite3`)
4. **Never use sync Redis** — use `redis.asyncio` only, never `redis.Redis`
5. **Never use `requests`** — use `httpx.AsyncClient` for all HTTP calls
6. **Wrap unavoidable sync calls** with `asyncio.to_thread()` — only for these 5 approved touch-points:
   - LangFuse SDK (no async version)
   - Gmail OAuth (inherently blocking)
   - SQLAlchemy DDL at startup
   - Cleanup job entry point
   - Pure CPU helpers with no I/O
7. **Do NOT add new sync touch-points** — check `docs/Mirix_async_native_changes.md` first

### Project-Specific Patterns

#### 1. Memory System Architecture
- **Meta Agent** orchestrates specialized memory agents
- **Sub-agents** extract specific memory types (episodic, semantic, etc.)
- **Memory extraction** happens via queue-worker architecture
- Each memory type has dedicated ORM models, schemas, and managers

#### 2. Database Patterns
- **ORM Models**: Located in `mirix/orm/` using SQLAlchemy
- **Schemas**: Located in `mirix/schemas/` using Pydantic
- **Managers**: Located in `mirix/services/` for business logic
- Always use `OrganizationMixin`, `UserMixin`, `AgentMixin` where applicable

#### 3. Agent Execution Flow
- `step()` method is the main agent execution loop (like LangChain's AgentExecutor)
- `inner_step()` handles single LLM interactions with tool calls
- `save_agent()` persists agent state to database
- Steps are logged to the `steps` table for audit/analytics (write-only)

#### 4. Message Flow
- Queue messages (transient) != Database messages (persistent)
- Database messages stored via `MessageManager.create()` after agent execution
- Messages link to steps via `step_id` foreign key
- Messages table used for conversation history and search

#### 5. API Patterns
- REST endpoints in `mirix/server/rest_api.py`
- Client SDK in `mirix/client/remote_client.py`
- Always authenticate with JWT or API key
- Use `create_or_get_user()` to ensure user exists before operations

#### 6. Settings Management
- Environment variables defined in `mirix/settings.py`
- Use Pydantic BaseSettings with `env` prefix `MIRIX_`
- Provider API keys in `ModelSettings`
- Tool configs in `ToolSettings`

#### 7. Tool Execution
- User-defined tools run in `ToolExecutionSandbox` for security
- Tools defined in `mirix/functions/function_sets/`
- Memory tools in `memory_tools.py`, base tools in `base.py`

### Code Review Checklist
Before suggesting changes, verify:
- [ ] No duplicate code or configurations
- [ ] Type hints are present and correct
- [ ] Error handling is specific and informative
- [ ] Docstrings follow PEP 257
- [ ] No hardcoded secrets or API keys
- [ ] Database operations use proper ORM patterns
- [ ] Async functions are properly awaited
- [ ] Tests are included for new functionality
- [ ] Logging uses appropriate levels (debug, info, warning, error)

### Common Pitfalls to Avoid
1. **Do NOT** confuse queue messages with database messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mirix-AI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
