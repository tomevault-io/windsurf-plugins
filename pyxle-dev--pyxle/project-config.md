---
trigger: always_on
description: This file covers technical rules for the Pyxle core framework.
---

# CLAUDE.md — Pyxle Core Framework

This file covers technical rules for the Pyxle core framework.
Every rule here exists to keep Pyxle enterprise-grade, stable, and maintainable.

---

## Project Overview

Pyxle is a Python-first full-stack web framework. `.pyxl` files colocate Python server
logic (`@server` loaders, `@action` mutations) with React/JSX components. The stack is
Starlette (ASGI), Vite (bundling), React 19 (rendering), and esbuild (SSR transpilation).

**Key files to read first:**
- `ROADMAP.md` — current phase, pending tasks, design principles
- `PYXLE_AUDIT.md` — architectural strengths, risks, and bottlenecks
- `pyproject.toml` — dependencies, test config, coverage thresholds
- `pyxle/runtime.py` — the `@server` and `@action` decorator contracts
- `pyxle/compiler/parser.py` — the `.pyxl` parser (most complex module)
- `pyxle/devserver/starlette_app.py` — request routing and middleware stack
- `pyxle/ssr/renderer.py` — SSR rendering pipeline (performance-critical)

---

## Mandatory Rules

### 1. Run Tests After Every Change

**This is non-negotiable.** Every code change must be followed by running the test suite.

```bash
# Run the full test suite
pytest

# The above command uses pyproject.toml defaults:
#   --strict-markers --strict-config
#   --cov=pyxle.build --cov=pyxle.cli --cov=pyxle.compiler --cov=pyxle.devserver --cov=pyxle.ssr
#   --cov-report=term-missing
```

- **Coverage threshold is 95%.** The build fails below this. Do not lower it.
- **All tests must pass.** Zero test failures are acceptable.
- **If you break a test, fix it before moving on.** Do not leave failing tests for later.
- **If you add a feature, add tests for it** in the same change. No feature ships without tests.

### 2. Write Tests First When Possible

Prefer test-driven development:
1. Write a failing test that describes the expected behavior
2. Implement the minimum code to make it pass
3. Refactor while keeping tests green

### 3. Never Skip or Weaken Tests

- DO NOT add `@pytest.mark.skip`, `pytest.mark.xfail`, or `# pragma: no cover` to dodge coverage
- DO NOT delete tests to make the suite pass
- DO NOT lower `fail_under = 95` in `pyproject.toml`
- DO NOT remove modules from the `--cov=` list

### 4. Test File Location Convention

Tests mirror the source tree:
```
pyxle/cli/           -> tests/cli/
pyxle/compiler/      -> tests/compiler/
pyxle/devserver/     -> tests/devserver/
pyxle/ssr/           -> tests/ssr/
pyxle/build/         -> tests/build/
pyxle/config.py      -> tests/test_config.py
```

When creating a new module at `pyxle/foo/bar.py`, create `tests/foo/test_bar.py`.

---

## Architecture Rules

### 5. Respect Module Boundaries

The codebase has clear separation of concerns:

| Module | Responsibility | May Import From |
|--------|---------------|-----------------|
| `pyxle/cli/` | CLI commands, user-facing I/O | Everything below |
| `pyxle/devserver/` | Dev server, Vite proxy, file watcher | compiler, ssr, routing, config |
| `pyxle/ssr/` | Server-side rendering, head merging | compiler (models only), config |
| `pyxle/compiler/` | `.pyxl` parsing, code generation | Nothing from pyxle (standalone) |
| `pyxle/routing/` | File-based route calculation | Nothing from pyxle (standalone) |
| `pyxle/build/` | Production build pipeline | compiler, devserver, config |
| `pyxle/config.py` | Configuration parsing | Nothing from pyxle (standalone) |
| `pyxle/runtime.py` | `@server`, `@action` decorators | Nothing from pyxle (standalone) |
| `pyxle/client/` | Client-side JS/JSX components | N/A (not Python -- JS only) |

**DO NOT** introduce circular imports. **DO NOT** have `compiler` depend on `devserver`.
**DO NOT** have `runtime.py` import anything from the framework -- it must stay zero-dependency
because it's injected into user code.

### 6. Frozen Dataclasses Everywhere

All data-carrying classes must be frozen dataclasses:

```python
# CORRECT
@dataclass(frozen=True)
class PageRoute:
    path: str
    module_key: str
    has_loader: bool

# WRONG -- mutable state causes bugs in async code
@dataclass
class PageRoute:
    path: str
    module_key: str
    has_loader: bool
```

Use `slots=True` for internal-only dataclasses that benefit from memory efficiency.

### 7. Use `Sequence` and `tuple` for Immutable Collections

```python
# CORRECT -- signals immutability
def process_routes(routes: Sequence[PageRoute]) -> tuple[str, ...]: ...

# WRONG -- signals mutability
def process_routes(routes: list[PageRoute]) -> list[str]: ...
```

Store collection fields as `tuple`, not `list`, in frozen dataclasses.

### 8. Async by Default

All I/O operations must be async. Never block the event loop.

If wrapping a synchronous call (like `subprocess.run`), use `asyncio.to_thread()`.

### 9. Use Structured Error Types

Every error that a user or developer might see needs a specific exception class.
Error classes live in the module they belong to (e.g., `pyxle/compiler/exceptions.py`).
Error messages must be specific, actionable, and include context (file path, line number, etc.).

### 10. No Magic, No Hidden Behavior

Decorators add metadata. They do NOT wrap, transform, or hide behavior.
The same principle applies to `@action` and any future decorators.

---

## Code Quality Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pyxle-dev/pyxle](https://github.com/pyxle-dev/pyxle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
