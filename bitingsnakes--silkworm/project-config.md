---
trigger: always_on
description: **Silkworm** is an async-first web scraping framework built on top of `wreq` (HTTP client with browser impersonation) and [scraper-rs](https://github.com/RustedBytes/scraper-rs) (fast HTML parsing). It provides a minimal Spider/Request/Response model, middlewares, and pipelines for building web scrapers and crawlers without boilerplate.
---

# Agent Guide for Silkworm

## Project Overview

**Silkworm** is an async-first web scraping framework built on top of `wreq` (HTTP client with browser impersonation) and [scraper-rs](https://github.com/RustedBytes/scraper-rs) (fast HTML parsing). It provides a minimal Spider/Request/Response model, middlewares, and pipelines for building web scrapers and crawlers without boilerplate.

### Key Features
- **Async-first engine** with configurable concurrency, bounded backpressure (defaults to `concurrency * 10`), and per-request timeouts
- **wreq-powered HTTP client** with browser impersonation, redirect following with loop detection, query merging, and proxy support
- **Typed spiders and callbacks** with `HTMLResponse` helpers (`follow`, selectors) and flexible callback outputs
- **Middleware system** for request/response processing
- **Pipeline system** for data export to various formats and destinations
- **Structured logging + crawl stats** via logly (`SILKWORM_LOG_LEVEL`, periodic + final summaries)

### Target Python Versions
- **Python 3.13+** (primary target; `pyproject.toml` requires `>=3.13,<3.15`)
- **Python 3.14** experimental support (including free-threaded build via `justfile-3.14t`)

## Python 3.13/3.14 Language Features and Best Practices

This project **requires** Python 3.13+ and makes extensive use of modern Python language features. When working on this codebase, you **MUST** use these features appropriately.

### PEP 695: Type Parameter Syntax (Python 3.12+)

**CRITICAL:** This project uses the `type` statement for type aliases. This is the preferred way to define type aliases in Python 3.12+.

#### ✅ Correct Usage (PEP 695)
```python
# From src/silkworm/_types.py
type JSONScalar = str | int | float | bool | None
type JSONValue = JSONScalar | dict[str, JSONValue] | list[JSONValue]
type Headers = dict[str, str]
type MetaData = dict[str, JSONValue]

# From src/silkworm/request.py
type CallbackOutput = (
    Request
    | JSONValue
    | Iterable[Request | JSONValue]
    | AsyncIterable[Request | JSONValue]
    | AsyncIterator[Request | JSONValue]
    | None
)
type CallbackResult = CallbackOutput | Awaitable[CallbackOutput]
type Callback = Callable[["Response"], CallbackResult]
```

#### ❌ Incorrect (Old Style - DO NOT USE)
```python
# Don't use TypeAlias annotation (pre-3.12 style)
from typing import TypeAlias
JSONScalar: TypeAlias = str | int | float | bool | None

# Don't use generic type parameters in classes without PEP 695
from typing import Generic, TypeVar
T = TypeVar('T')
```

**When to use:** Always prefer `type` statements for type aliases in this codebase.

### PEP 636: Structural Pattern Matching (Python 3.10+)

The project uses pattern matching for cleaner conditional logic.

#### ✅ Correct Usage
```python
# From src/silkworm/middlewares.py
match self._strategy:
    case _DelayStrategy.CUSTOM:
        assert self._delay_func is not None
        delay = self._delay_func(request, spider)
    case _DelayStrategy.RANDOM:
        assert self._min_delay is not None and self._max_delay is not None
        delay = random.uniform(self._min_delay, self._max_delay)
    case _DelayStrategy.FIXED:
        assert self._fixed_delay is not None
        delay = self._fixed_delay
    case _:
        assert_never(self._strategy)
```

**When to use:** Use pattern matching for multi-way conditionals based on enums, types, or structured data. Prefer `match` over long `if/elif` chains when checking multiple cases.

### PEP 604: Union Type Operator (Python 3.10+)

**CRITICAL:** Always use the `|` operator for union types instead of `typing.Union`.

#### ✅ Correct Usage
```python
from __future__ import annotations

def process(value: str | int | None) -> Response | Request:
    ...

class Spider:
    def __init__(self, name: str | None = None) -> None:
        ...
```

#### ❌ Incorrect (Old Style - DO NOT USE)
```python
from typing import Union, Optional

def process(value: Union[str, int, None]) -> Union[Response, Request]:
    ...

# Don't use Optional
def __init__(self, name: Optional[str] = None) -> None:
    ...
```

**Important:** Always include `from __future__ import annotations` at the top of every module for forward references and cleaner type hints.

### PEP 585: Type Hinting Generics in Standard Collections (Python 3.9+)

**CRITICAL:** Use built-in collection types directly for type hints, not `typing` module equivalents.

#### ✅ Correct Usage
```python
from collections.abc import AsyncIterator, Iterable, Callable

def process_urls(urls: list[str]) -> dict[str, int]:
    ...

async def generate_items() -> AsyncIterator[dict[str, str]]:
    ...

class Spider:
    start_urls: tuple[str, ...] = ()
```

#### ❌ Incorrect (Old Style - DO NOT USE)
```python
from typing import List, Dict, Tuple

def process_urls(urls: List[str]) -> Dict[str, int]:
    ...
```

**When to use:** Always use `list`, `dict`, `tuple`, `set` directly for type hints. Import abstract types from `collections.abc`, not `typing`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BitingSnakes/silkworm](https://github.com/BitingSnakes/silkworm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
