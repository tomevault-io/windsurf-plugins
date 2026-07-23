---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

fastapi-chameleon adds Chameleon template-language support to FastAPI: call `global_init(template_folder)` once at startup, decorate view functions with `@template(...)`, and return a dict of template variables. It ships as the `fastapi_chameleon` package (PyPI: `fastapi-chameleon`). Runtime dependencies are exactly `fastapi` and `chameleon`; Python 3.10+ required.

## Commands

A project venv exists at `./venv` (uv-managed CPython 3.14); all VS Code tasks and docs tooling point at it.

```bash
uv pip install -r requirements-dev.txt   # dev setup (the venv is uv-managed and has NO pip — plain `pip install` fails)

pytest                                # run tests (no pytest config; plain discovery from repo root)
pytest tests/test_render.py::test_can_decorate_dict_async_method   # single test
ruff check .                          # lint (config: ruff.toml — 120 cols, single quotes, E/F)
ruff format .                         # format
venv/bin/ty check fastapi_chameleon   # type-check (pyrefly is also installed but not yet configured)

venv/bin/python scripts/build_docs.py # build docs site (default VS Code build task; must use the venv python — it finds great-docs next to sys.executable)
venv/bin/great-docs preview           # live docs preview
venv/bin/python scripts/serve_docs.py # production-faithful preview at http://127.0.0.1:8099/docs/fastapi-chameleon/

python example/example_app.py         # demo app on 127.0.0.1:8000 (must run as a script — global_init() is inside main(), so the uvicorn CLI won't work)
```

Packaging uses the hatchling backend; release with `uv build`, then `uv publish`.

## Architecture

The entire library is three modules in `fastapi_chameleon/` (plus a `py.typed` marker — the package ships its types to consumers), with all logic in `engine.py`:

- **Module-global engine state** (`engine.py`): `global_init()` populates a private module-level `PageTemplateLoader` and `template_path`. Repeat calls are no-ops unless `cache_init=False`. `engine.clear()` resets the state (not in `__all__`, but tests depend on it for isolation).
- **`@template` decorator**: works bare (`@template`) or parameterized (`@template('home/index.pt', mimetype=...)`) — distinguished by a `callable(template_file)` check. `_decorate()` picks a sync or async wrapper once at decoration time via `inspect.iscoroutinefunction`. When no template name is given, it derives `{last module segment}/{function name}` and tries `.html` first, falling back to `.pt` — and that existence check runs at decoration (import) time. Gotcha: if `global_init()` hasn't run yet when views are decorated, `template_path` silently defaults to `'templates'` (relative to CWD) rather than raising, and a later `global_init()` will not re-resolve already-derived names.
- **View return contract** (`__render_response`): a `dict` is rendered as template variables; a `fastapi.Response` passes through untouched (redirects etc.); anything else raises `FastAPIChameleonException`.
- **Error pages are exception-based control flow**: `not_found()` and `generic_error()` always raise (`FastAPIChameleonNotFoundException` / `FastAPIChameleonGenericException` from `exceptions.py`), so they work from anywhere below a decorated view. The decorator wrappers catch them and render the error template — always as `text/html`, ignoring the decorator's `mimetype`. The two paths are asymmetric: `generic_error()` forwards `template_data` into the template, but `not_found()` always renders the 404 template with an empty context.
- **Typing is deliberate**: `template()` has ParamSpec-based `@overload`s, and `_decorate()` returns the wrapper through `cast(Callable[P, R], ...)` even though the wrappers really return `fastapi.Response`. This keeps the decorator signature-transparent for FastAPI dependency injection and pyright — preserve that property when touching `engine.py`.

Public API (`__init__.py` `__all__`): `template`, `global_init`, `not_found`, `response`, `generic_error`.

## Tests

Tests call decorated view functions directly (via `asyncio.run()` for async) and assert on the returned `fastapi.Response` — no TestClient or HTTP layer. Because engine state is module-global, most tests use the `setup_global_template` fixture in `tests/conftest.py` (calls `global_init` on `tests/templates/`, then `fc.engine.clear()` on teardown). A test that calls `global_init()` itself must pass `cache_init=False` (repeat init is otherwise a no-op) and clear state afterward, or it leaks into tests that expect an uninitialized engine (see `tests/test_init.py`). The `tests/templates/test_render/` folder exists specifically to exercise the default-template-name convention (`{test module name}/{function name}`).

## Documentation Pipeline

Docs are built with Great Docs (Quarto-based) in a three-layer flow; getting this wrong wastes work:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikeckennedy/fastapi-chameleon](https://github.com/mikeckennedy/fastapi-chameleon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
