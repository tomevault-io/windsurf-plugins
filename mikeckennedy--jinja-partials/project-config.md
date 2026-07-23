---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

jinja_partials (PyPI: `jinja-partials`) is a small, fully typed Python library that adds a `render_partial()` global to Jinja2 templates so HTML fragments ("partials") can be composed and nested with explicitly passed data. It works with plain Jinja2 and has first-class registration helpers for Flask, Quart, FastAPI, and Starlette. Requires Python >= 3.10; the only runtime dependency is `jinja2`.

The entire library is one module: `jinja_partials/__init__.py`, plus the empty `py.typed` marker (PEP 561) that must remain in the package and ship in the wheel.

## Commands

The project virtualenv is `venv/` (not `.venv/`) and is uv-managed — it has **no pip**. Use `uv pip ...` for installs; tools live in `venv/bin/`.

```bash
venv/bin/pytest                                # run the test suite (all tests live in tests/test_rendering.py)
venv/bin/pytest tests/test_rendering.py::test_render_with_data   # run a single test by node id
ruff check . && ruff format .                  # lint + format (ruff.toml: 120 cols, single quotes, rules E/F/I)
uv build                                       # build sdist/wheel via hatchling (`python -m build` fails: build pkg not installed)
uv pip install -r requirements-development.txt # install dev deps (pytest, all four frameworks, great-docs)
venv/bin/python scripts/build_docs.py          # build the docs site into docs/ (the default VS Code build task)
venv/bin/python scripts/serve_docs.py          # preview committed docs/ at http://127.0.0.1:8099/docs/jinja-partials/
```

Dependency changes: edit the `.piptools` source files (`requirements.piptools` = runtime, `requirements-development.piptools` = dev), then recompile the pinned lockfiles — never hand-edit the compiled `.txt` files:

```bash
uv pip compile requirements-development.piptools --output-file requirements-development.txt --exclude-newer "1 week"
```

pytest config (`pytest.ini`) runs strict: `--strict-markers --strict-config`, and `filterwarnings = error` — any warning other than `UserWarning`/`DeprecationWarning` fails the suite, and new markers must be declared in the ini.

## Architecture

### Layering: one primitive, framework adapters around it

- `render_partial(template_name, renderer=None, markup=True, **data)` is the core primitive. With no renderer it falls back to `flask.render_template`, raising `PartialsException` if Flask isn't installed.
- `generate_render_partial(renderer, markup=...)` is just `functools.partial(render_partial, ...)` binding a renderer.
- Every `register_*` function builds a framework-specific synchronous `renderer(template_name, **data) -> str` closure and installs it as a Jinja global via `env.globals.update(render_partial=generate_render_partial(renderer, ...))`. Templates then call `{{ render_partial('shared/partials/x.html', model=value) }}`, including nested/recursive partials.
- Registration paths: `register_extensions(app)` (Flask), `register_quart_extensions(app)`, `register_fastapi_extensions(app, templates)`, `register_starlette_extensions(templates, app=None)` (note: templates first, app as keyword), `register_environment(env)` (plain Jinja2), and the declarative `PartialsJinjaExtension` (`Environment(extensions=[...])`).
- Only the Flask `register_extensions` path renders through `flask.render_template`, so Flask context processors, `g`, and `request` are available inside partials there and **nowhere else** (including the `PartialsJinjaExtension` path on a Flask app).
- `markup` defaults differ by design: `render_partial`/`generate_render_partial` default `markup=True`, `register_environment` defaults `markup=False`, and all framework paths plus `PartialsJinjaExtension` hardcode `markup=True`. The `@overload` sets encode this (`Literal[True] -> Markup`, `Literal[False] -> str`) — changing a default is a typed behavior break.

### Async rendering: the executor machinery

Jinja template expressions cannot `await`, but Quart/FastAPI/Starlette use `enable_async=True` environments. The bridge: the renderer submits `asyncio.run(template.render_async(**data))` to a `ThreadPoolExecutor` worker thread (fresh event loop) and blocks on `future.result()`. This block-in-place is the accepted tradeoff — do not "fix" it by making the renderer async.

Three executor tiers:
1. A **dedicated executor** created inside the app's lifespan: FastAPI/Starlette wrap `app.router.lifespan_context` (executor stored on `app.state.jinja_partials_executor`); Quart uses `before_serving`/`after_serving` (stored in `app.extensions['jinja_partials_executor']`).
2. A lazy per-registration **fallback executor** (closure variable) for renders outside a serving cycle — intentionally never shut down.
3. A module-level `_async_render_executor` used by `register_environment` / `PartialsJinjaExtension` / app-less Starlette — also intentionally never shut down.

Lifecycle invariants to preserve (from "exception-safe and re-entrant" work, commit 20c0e11):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikeckennedy/jinja_partials](https://github.com/mikeckennedy/jinja_partials) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
