---
trigger: always_on
description: rekal is a Model Context Protocol (MCP) server that gives LLMs persistent long-term memory.
---

# rekal — Agent Instructions

## What this project is

rekal is a Model Context Protocol (MCP) server that gives LLMs persistent long-term memory.
It uses hybrid search (FTS5 + vector + recency) in a single SQLite file.
Python 3.11+, installed via `pip install rekal`, runs as a stdio MCP server.

## Architecture

```
MCP Client
  │ stdio (JSON-RPC)
  │
  mcp_adapter.py          ← FastMCP server, lifespan (creates/closes DB)
  │
  ├── tools/core.py       ─┐
  ├── tools/introspection.py│─ thin @mcp.tool() wrappers, delegate to SqliteDatabase
  ├── tools/smart_write.py  │
  └── tools/conversations.py┘
                            │
                    sqlite_adapter.py ← SqliteDatabase @dataclass, ALL SQL lives here
                            │
                            ├── SQLite (memories, conversations, tags, conflicts)
                            ├── FTS5 (full-text index, auto-synced via triggers)
                            └── sqlite-vec (vector index)
```

### Key rules

- `SqliteDatabase` is a `@dataclass` holding the `aiosqlite.Connection` and ALL query methods. Every SQL statement lives here. No SQL in tool files.
- Tool modules in `adapters/tools/` are thin wrappers. They register MCP tools via `@mcp.tool()` and call methods on `SqliteDatabase`.
- `mcp_adapter.py` creates the FastMCP server, manages lifespan, and imports tool modules so they register.
- Adding a new tool = add a method to `SqliteDatabase` + add a thin tool wrapper in the appropriate `tools/*.py` file.
- **No dynamic SQL.** SQL strings must be static literals. No f-strings, no string concatenation, no `%` formatting to build queries. Use subqueries and parameterized `?` placeholders instead.

## Python style — modern, strict, no shortcuts

### Never use `Any`

Zero tolerance. Every value has a real type. If you reach for `Any`, find the actual type.
Use `TYPE_CHECKING` imports, union types, `Protocol`, or type aliases instead.

```python
# Correct
SqlParam = str | int | float | bytes | None
params: list[SqlParam] = []

# Wrong
params: list[Any] = []
```

### No `getattr` or `setattr`

Never use `getattr()` or `setattr()`. Access attributes directly by name.
These hide attribute access from type checkers and make code harder to follow.

### No underscore prefixes on attributes or methods

Public by default. No `self._db`, no `_helper()`, no `_CONSTANT`.
Use plain names: `self.db`, `helper()`, `SCHEMA`.

```python
# Correct
@dataclass
class SqliteDatabase:
    db: aiosqlite.Connection
    embed: EmbeddingFunc

# Wrong
class SqliteDatabase:
    def __init__(self, db, embed):
        self._db = db
        self._embed = embed
```

### No mutable globals

No module-level mutable state. Constants like `SCHEMA` (a frozen SQL string) and type aliases like `SqlParam` are fine.
For values that might look like globals, use functions or dataclass fields instead.

```python
# Correct
def default_db_path() -> str:
    return str(Path.home() / ".rekal" / "memory.db")

# Wrong
_DEFAULT_DB_PATH = str(Path.home() / ".rekal" / "memory.db")
```

### Use dataclasses everywhere possible

Prefer `@dataclass` over hand-written `__init__`. Pydantic `BaseModel` only where serialization is needed (models.py).

### Use `Annotated` with `Field(description=...)` on all MCP tool parameters

Every tool parameter (except `ctx`) must have a description for the MCP schema.

```python
@mcp.tool()
async def memory_store(
    ctx: Context,
    content: Annotated[str, Field(description="The text content to store")],
    limit: Annotated[int, Field(description="Maximum number of results")] = 10,
) -> str:
```

### Type annotations on everything

Every function parameter, return type, and non-obvious variable. Use `from __future__ import annotations` in every file.

### Use union syntax, not Optional

```python
# Correct
project: str | None = None

# Wrong
project: Optional[str] = None
```

## Testing

### 100% coverage, no exceptions

Every line tested. If truly untestable (e.g., defensive except for impossible conditions), add `# pragma: no cover` with explanation.

### No mocking, no monkey patching

Use real implementations:
- Real in-memory SQLite via `:memory:` (sub-millisecond, tests real SQL)
- Deterministic hash-based embeddings from `conftest.py` (fast, deterministic, correct shape)
- Real MCP tool functions called with constructed context objects

### Test structure

```
tests/
├── conftest.py                  # db fixture (in-memory), deterministic_embed
├── test_sqlite_adapter.py       # Direct DB operations
├── test_search.py               # Hybrid search scoring
├── test_introspection.py        # memory_similar, topics, timeline, etc.
├── test_smart_write.py          # supersede, build_context
├── test_conversations.py        # Conversation DAG operations
├── test_scoring.py              # Score normalization math
├── test_embeddings.py           # Embedding utilities
├── test_core_tools.py           # MCP tool wrappers for core
├── test_introspection_tools.py  # MCP tool wrappers for introspection
├── test_smart_write_tools.py    # MCP tool wrappers for smart_write
├── test_conversation_tools.py   # MCP tool wrappers for conversations
├── test_mcp_adapter.py          # Lifespan test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [janbjorge/rekal](https://github.com/janbjorge/rekal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
