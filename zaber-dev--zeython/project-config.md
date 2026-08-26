---
trigger: always_on
description: generates list/create/edit/delete pages for registered models from
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Copilot Workspace, etc.)
working in a Zeython codebase — either the framework itself, or an application
built with it.

## What Zeython is

An async-first MVC framework: ASGI (Starlette) + async SQLAlchemy 2.0, a
dependency injection container, a service-provider boot lifecycle, and a
`zeython` CLI. Full details: [docs/architecture.md](docs/architecture.md).

## If you have MCP tool access

Zeython ships its own MCP server (`zeython mcp`, requires the `mcp` extra).
If it's connected, prefer `list_routes`/`list_models`/`app_info` over
grepping `routes/web.py` or `app/Models/*.py` by hand — they reflect the
project's actual registered routes and mapped schema, not what the source
*looks like* it does. Prefer `search_docs` over recalling a Zeython API
from training data; it searches the docs bundled with the exact installed
framework version. See [docs/ai-agents.md](docs/ai-agents.md).

## If you're working in the framework itself (`src/zeython/`)

- Read `docs/architecture.md` first — it explains the container/provider/router
  relationship; don't guess at it from file names.
- Every public API needs type hints. `mypy src/zeython` must pass with zero
  errors before you're done.
- Add or update tests in `tests/` for any behavior change. `pytest` must pass.
- Run `ruff check src tests` — fix, don't suppress, unless there's a real
  reason (document it with a `# noqa: CODE` and a one-line comment why).
- `src/zeython/cli/templates/starter/` is copied verbatim by `zeython new` and
  is **not** type-checked or linted as part of the package (see the `exclude`
  entries in `pyproject.toml`) — those files run in a *generated project's*
  context, not the framework's. If you touch them, verify by hand:
  ```bash
  pip install -e .
  zeython new "Agent Smoke Test" --path /tmp/agent-smoke-test
  cd /tmp/agent-smoke-test && cp .env.example .env && pip install -e ".[dev]"
  python -m alembic revision --autogenerate -m x && python -m alembic upgrade head
  pytest
  ```
- Don't add a dependency to `pyproject.toml` for something solvable in a few
  lines of stdlib. The framework's value is a small, trustworthy core.

## If you're working in an application built with Zeython

- Models go in `app/Models/`, subclass `zeython.Model`, and get registered in
  `app/Models/__init__.py` (so Alembic autogenerate can see them). Use
  `zeython make model <Name>` rather than hand-rolling this.
- Relationships are plain SQLAlchemy `relationship()` — no framework wrapper
  needed to define one. Loading is where it matters: NEVER touch a
  relationship attribute you didn't fetch with `include=("name",)` (on
  `find`/`all`/`find_by`/`first_where`) — it raises `MissingGreenlet` in
  async code, not a normal lazy load. Same rule for `to_dict()`: pass
  `include=` only for relationships you eager-loaded, or it raises a clear
  `RuntimeError` telling you so (better than `MissingGreenlet`, still an
  error). See `docs/relationships.md`.
- Register `N1QueryDetectionServiceProvider` (`zeython.n_plus_one`) in dev
  to catch a forgotten `include=` before it ships -- it warns
  (`APP_DEBUG`-only, safe to always register) when a request fires the
  same query shape suspiciously many times, the sign of fetching a
  relationship one row at a time instead of eager-loading it. See
  `docs/relationships.md#detecting-n1s-automatically`.
- Validate on the model, not in the controller: set `__rules__` on the model
  (`docs/validation.md`) instead of hand-checking `request.json()` fields —
  `create()`/`save()`/`update()` already raise `ValidationException` (a 422
  JSON response) for you.
- For behavior that should always run around a save or delete (normalizing
  a field, deriving one, cache invalidation), override the model's
  lifecycle hooks (`creating`/`created`/`updating`/`updated`/`deleting`/
  `deleted`, plus `saving`/`saved` for both create and update) rather than
  repeating that logic in every controller that touches the model.
  `creating()`/`updating()` run *before* `__rules__` validation, so they're
  the right place to derive a field validation then checks. See
  `docs/model-events.md`.
- Server-rendered HTML goes through `zeython.views.render(request, name,
  context)`, reading from `resources/views/` — register `ViewServiceProvider`
  in `main.py` first (`docs/views.md`). Don't hand-roll Jinja2 environments.
- Auth is session-based (`docs/authentication.md`): mix `Authenticatable`
  into your user model for `set_password()`/`check_password()`, register
  `AuthServiceProvider(app, user_model=User)`, and guard routes with
  `await require_auth(request)` (raises 401) or `await current_user(request)`
  (returns `None`). Never hand-roll password hashing — use
  `zeython.hash_password`/`verify_password` (PBKDF2-HMAC-SHA256), and never
  log or serialize a password/hash field — put it in the model's `__hidden__`.
- `AuthServiceProvider` turns on CSRF protection automatically (`docs/csrf.md`)
  — every `POST`/`PUT`/`PATCH`/`DELETE` against a session-authenticated
  route needs the `X-CSRF-Token` header matching the `csrf_token` cookie.
  Don't turn `CSRF_ENABLED` off to make a request "just work" — fix the
  client (read the cookie, send the header) instead; in tests,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zaber-dev/Zeython](https://github.com/zaber-dev/Zeython) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
