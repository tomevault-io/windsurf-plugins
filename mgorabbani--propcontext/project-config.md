---
trigger: always_on
description: Cross-tool memory file for Claude Code, Cursor, Codex, and any other agent. Source of truth for design pattern and code style. **Keep current.** When patterns change, update this file in the same commit.
---

# AGENTS.md — PropContext

Cross-tool memory file for Claude Code, Cursor, Codex, and any other agent. Source of truth for design pattern and code style. **Keep current.** When patterns change, update this file in the same commit.

> `CLAUDE.md` is a symlink to this file. Do not duplicate.

---

## What this repo is

Living building memory for property management (German WEG domain). Pipeline ingests emails / invoices / bank tx / `stammdaten.json`, compresses into one `building.md` per property. External AI agent fetches `building.md` over HTTP. Full PRD: `PRD_Overview`. Hackathon scope: 48h MVP.

## Tech stack (2026-04 baseline)

| Layer | Choice | Version |
|---|---|---|
| Runtime | Python | 3.13+ |
| Pkg/env | uv | 0.11+ |
| API | FastAPI + uvicorn | 0.136.1 |
| Validation | Pydantic v2 | 2.13+ |
| Settings | pydantic-settings | 2.14+ |
| Logging | structlog | 25+ |
| HTTP client | httpx (async) | 0.28+ |
| Async fs | anyio | 4.7+ |
| Lint+format | ruff | 0.15+ |
| Type check | ty (Astral) | 0.0.32+ |
| Test | pytest + pytest-asyncio | 9 / 1.3 |
| Container | Docker (multi-stage, uv) | 29+ |

Do not add libraries without first comparing 2-3 alternatives in chat and getting human sign-off. The ecosystem moves fast; "the classic choice" is usually wrong.

## Layout

```
app/
  main.py              # create_app(), lifespan, mounts /mcp
  api/v1/
    router.py          # mounts feature routers
    health.py
    buildings.py
  core/
    config.py          # Settings, get_settings (lru_cache)
    logging.py         # configure_logging
  mcp/                 # FastMCP server exposed at /mcp (org-scoped, OAuth via WorkOS)
    server.py          # build_mcp(settings) → FastMCP
    auth.py            # AuthKitProvider wiring
    context.py         # current_org_id, assert_property_access (per-call)
    orgs.py            # hardcoded org → property allowlist
    tools.py / resources.py / prompts.py
  schemas/             # Pydantic response/request models, no business logic
  services/            # business logic, IO, integrations
tests/
  conftest.py          # AsyncClient fixture, dependency_overrides
  test_*.py
data/                  # raw source dataset (read-only)
schema/                # extraction prompts + WIKI_SCHEMA
output/                # building.md files (written by pipeline, served by API)
```

Rule: `api/` knows about `services/`, never the reverse. `services/` knows about external IO + `core/`. `schemas/` is leaf — imports nothing project-internal except other schemas.

## Code style (non-negotiable)

1. **`from __future__ import annotations` at the top of every `.py`** (except `__init__.py` if empty).
2. **Modern types only.** `list[str]` not `List[str]`. `X | None` not `Optional[X]`. `dict[K, V]` not `Dict`. No `typing.Tuple/List/Dict/Optional/Union` imports.
3. **Async by default for routes and IO services.** Blocking IO inside async = bug. Use `anyio` (or `asyncio.to_thread` for CPU-light blocking calls). No `requests` — `httpx.AsyncClient` only.
4. **Lifespan, not `@app.on_event`.** `on_event` is deprecated.
5. **Annotated DI.** Routes take `param: Annotated[T, Depends(provider)]`. Never bare `Depends()` in default arg.
6. **Pydantic v2 idioms.** `model_config = SettingsConfigDict(...)` / `ConfigDict(...)`. No nested `class Config`. Use `Field(default=...)`, not mutable defaults.
7. **Settings are immutable + cached.** One `get_settings()` `@lru_cache(maxsize=1)` factory. Routes get `Settings` only via `Depends(get_settings)` — never import the singleton.
8. **Structured logs only.** `log = structlog.get_logger(__name__)`. `log.info("event", key=value)`. No f-strings in log messages, no `print`.
9. **Errors raise `HTTPException`** with explicit status. Don't return error dicts.
10. **One `create_app()` factory.** Module-level `app = create_app()` is the ASGI entrypoint. No global mutation after import.
11. **Path validation at the edge.** Building IDs / user input go through Pydantic constraints (`Path(pattern=...)`, `StringConstraints`) before touching the filesystem. Never `Path(user_input)` raw.
12. **No comments stating *what*** the code does. Comment only the non-obvious *why*. Docstrings on public service classes/functions only.

## Patterns

### Adding a route

1. New module in `app/api/v1/<feature>.py` exposing `router = APIRouter()`.
2. Register in `app/api/v1/router.py` with `prefix=` and `tags=`.
3. Pydantic response model in `app/schemas/<feature>.py`.
4. Business logic in `app/services/<feature>.py` with a `get_<feature>_service()` Depends provider.
5. Test in `tests/test_<feature>.py` using the `client` fixture.

### Adding a setting

1. Field on `Settings` in `app/core/config.py` with type + default.
2. Document in `.env.example` (create when first needed).
3. Override in tests via the `settings` fixture / `dependency_overrides`.

### Adding a dependency

```bash
uv add <pkg>            # runtime
uv add --group dev <pkg> # dev/test
```

Pin a sane lower bound (`>=X.Y`). Never pin exact (`==`) except for security CVEs.

### Adding an MCP capability


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mgorabbani/PropContext](https://github.com/mgorabbani/PropContext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
