---
trigger: always_on
description: This file describes the conventions and design principles for contributing to beans.
---

# Beans — Agent Guide

This file describes the conventions and design principles for contributing to beans.
Follow these closely — they are intentional and reflect the project's values.

## Philosophy

- **Low cognitive load.** Code reads top-to-bottom without jumping around.
- **Pure by default.** Push I/O to the boundary. Models and helpers are pure functions.
- **No over-engineering.** Don't build abstractions until you have two concrete uses.
- **Defer what you don't need.** If a feature isn't needed yet, don't add it.

## Architecture

```
src/beans/
├── models.py    # Pydantic models, pure functions, no I/O
├── store.py     # SQLite I/O boundary, BeanStore class
├── api.py       # Command API, composes store calls, returns models
├── config.py    # Global config (~/.config/beans/config.json)
├── project.py   # Project discovery (find .beans/, init)
└── cli.py       # Typer CLI, thin wiring layer (parse → call → format)
```

- **models.py** — Pure data and pure functions. No imports from store or cli.
- **store.py** — The only place that touches the database. Accepts injected connections.
- **api.py** — Command API. Each function is one use case, composes store calls.
- **config.py** — Global user config. Reads from XDG config directory.
- **project.py** — Project discovery. Finds `.beans/` dir, handles `beans init`.
- **cli.py** — Thin wiring. No business logic. Display formatting lives here.

## Code Style

### No underscore prefixes

Everything is public. Don't use `_` to signal "private" — if it's in the module, it's
part of the module.

### Functions over methods

If it doesn't need `self`, it's a standalone function, not a method. Compose small
functions rather than building class hierarchies.

```python
# Yes
def columns(cursor):
    return [desc[0] for desc in cursor.description]

def row(cols, values):
    return dict(zip(cols, values))

# No
class BeanStore:
    def _get_columns(self, cursor): ...
    def _row_to_dict(self, cols, values): ...
```

### Default arguments for configurability

Never reference module-level constants directly inside function bodies. Instead, pass
them as default arguments. This makes functions testable and reusable without adding
configuration infrastructure. For classes, use class attributes instead of reaching for
the global.

```python
# Yes — constant flows through the signature
def find_beans_dir(start=None, dirname=BEANS_DIR) -> Path: ...

# Yes — class owns its configuration
class BeanId(str):
    pattern = ID_PATTERN
    def __new__(cls, value="", **kwargs):
        if not cls.pattern.match(value): ...

# No — function reaches for the global directly
def find_beans_dir(start=None) -> Path:
    candidate = current / BEANS_DIR  # hidden dependency
```

### `**kwargs` over dict for named fields

When a function accepts a set of named fields, use `**kwargs` instead of a dict parameter.
It reads better at the call site and lets the interpreter catch typos.

```python
# Yes
def update(self, bean_id, **fields) -> int: ...
store.update(bean_id, title="New title", status="closed")

# No
def update(self, bean_id, fields: dict) -> int: ...
store.update(bean_id, {"title": "New title", "status": "closed"})
```

### Constants for magic values

```python
ID_BYTES = 4
```

### Let Pydantic handle coercion

Don't manually convert what Pydantic validates and coerces automatically. For example,
Pydantic coerces ISO strings to datetime — don't call `fromisoformat()` yourself.

### Import organization

Use section comments and `force-sort-within-sections`:

```python
# Python imports
import json
import sqlite3

# Pip imports
import typer

# Internal imports
from beans.models import Bean
```

### Type annotations

Annotate return types. Don't annotate parameters when the default value tells the story.

## Dependency Injection

### Inject, don't construct

BeanStore takes a `sqlite3.Connection`, not a path. Factory classmethods handle
construction. This enables testing with `:memory:` databases.

```python
# Production
store = BeanStore.from_path("beans.db")

# Testing
store = BeanStore(sqlite3.connect(":memory:"))
```

### Context manager protocol

Resource-holding classes implement `__enter__`/`__exit__` to avoid explicit `close()`.

```python
with BeanStore.from_path("beans.db") as store:
    store.create(bean)
```

### Model stays pure, display lives in CLI

Don't add `__str__` or `__format__` to models for display purposes. Display formatting
is a CLI concern — use standalone functions like `format_bean()`.

## Testing

### No mocks

BeanStore gets a real SQLite `:memory:` database. Test real behavior, not mock wiring.

### Assert against the model, not individual fields

Leverage Pydantic equality to compare whole objects. Don't decompose into field-by-field
checks when a single comparison says it all.

```python
# Yes — one assertion, full structural check
assert store.get(bean.id) == bean
assert store.list() == [b1, b2]

# No — decomposing what equality already covers
result = store.get(bean.id)
assert result.id == bean.id
assert result.title == "Fix auth"
```

For model defaults, use `model_dump()` against an expected dict with fixed `id` and
`created_at` to pin dynamic fields.

```python
def test_defaults(self):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henriquebastos/beans](https://github.com/henriquebastos/beans) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
