---
trigger: always_on
description: > Context file for AI coding assistants. Symlinked as `CLAUDE.md`.
---

# AGENTS.md — ToolRegistry

> Context file for AI coding assistants. Symlinked as `CLAUDE.md`.

## What this project is

ToolRegistry is a **protocol-agnostic tool management library** for
function-calling LLMs. It unifies tool registration, schema generation, and
execution across multiple LLM providers (OpenAI, Anthropic, Gemini) and
external ecosystems (MCP, OpenAPI, LangChain).

This is the **core** package in a three-package ecosystem:

| Package | Role | Depends on |
|---------|------|------------|
| `toolregistry` (this) | Core library: `Tool` model, `ToolRegistry`, client integrations | — |
| `toolregistry-server` | Server infrastructure: `RouteTable`, OpenAPI/MCP adapters, auth, CLI | `toolregistry` |
| `toolregistry-hub` | Tool implementations + default server configuration | `toolregistry-server` |

Downstream packages are notified on release via `notify-downstream.yml`.

## Architecture

ToolRegistry is organized around a central `ToolRegistry` class composed of
7 orthogonal mixins:

| Mixin | File | Responsibility |
|-------|------|----------------|
| RegistrationMixin | `_mixins/registration.py` | Register/unregister tools from functions, classes, MCP, OpenAPI |
| EnableDisableMixin | `_mixins/enable_disable.py` | Enable/disable tools, track reasons, metadata updates |
| NamespaceMixin | `_mixins/namespace.py` | Namespace prefixing, merge, spinoff sub-registries |
| PermissionsMixin | `_mixins/permissions.py` | Tag-based permission policies, sync/async handlers |
| ExecutionLoggingMixin | `_mixins/logging.py` | In-memory call history (name, args, result, duration) |
| AdminMixin | `_mixins/admin.py` | Web admin panel lifecycle |
| ChangeCallbackMixin | `_mixins/callbacks.py` | Event subscriptions (register, enable, error, etc.) |

Key standalone modules:

| Module | Purpose |
|--------|---------|
| `tool.py` | `Tool`, `ToolMetadata`, `ToolTag`, `TOOLCALL_REASON_PROPERTY` |
| `tool_discovery.py` | BM25-based fuzzy tool search (`ToolDiscoveryTool`) |
| `parameter_models.py` | JSON Schema generation from Python signatures |
| `_rosetta.py` | Multi-provider schema bridge (via `llm-rosetta`) |
| `truncation.py` | Result truncation with temp file persistence |
| `admin/` | HTTP server, REST handlers, execution log, auth, static assets |
| `executor/` | Thread/process pool execution backends |
| `permissions/` | Policy engine, handler protocol, built-in rules |
| `config/` | Declarative JSONC/YAML config loader |
| `integrations/` | Native, MCP, OpenAPI, LangChain integration packages |

## Repository layout

```
src/toolregistry/
├── __init__.py              # Exports, __version__
├── tool.py                  # Tool, ToolMetadata, ToolTag
├── tool_registry.py         # ToolRegistry (composes 7 mixins)
├── tool_discovery.py        # BM25 fuzzy search
├── parameter_models.py      # JSON Schema from signatures
├── _rosetta.py              # llm-rosetta bridge
├── truncation.py            # Result size management
├── events.py                # ChangeEvent types
├── utils.py                 # Tool name normalization, etc.
├── _mixins/                 # 7 mixin classes (see Architecture)
├── admin/                   # Web admin panel (server, handlers, UI)
│   ├── server.py            # Async HTTP server
│   ├── handlers.py          # REST API endpoints
│   ├── admin.html           # Single-file SPA (i18n: EN/ZH)
│   ├── execution_log.py     # Call history
│   ├── auth.py              # Token auth
│   └── static.py            # Asset serving
├── executor/                # Pluggable execution backends
├── permissions/             # Policy engine + built-in rules
├── config/                  # JSONC/YAML config loader
├── integrations/            # MCP, OpenAPI, LangChain, native
├── types/                   # Provider-specific schema types
│   ├── common.py            # API_FORMATS, ToolCall, ToolCallResult
│   └── openai/, anthropic/, gemini/
└── _vendor/                 # Vendored zero-dep modules (DO NOT EDIT)

tests/                       # 39 test files, pytest
examples/                    # Usage examples
docs_en/, docs_zh/           # Documentation (git worktrees, orphan branches)
```

## Setup and commands

```bash
conda activate toolregistry
pip install -e ".[dev,test,mcp]"
pre-commit install
```

Run `make help` for all targets. Key ones:

```bash
make lint          # ruff check + ty check + complexipy
make fmt           # ruff check --fix + ruff format
make test          # pytest tests/ -v
make build         # python -m build
make push          # twine upload
```

Tooling config (ruff, ty, complexipy) lives in `pyproject.toml`.

## Definition of done

1. `pre-commit run --all-files` passes (ruff, ruff-format, ty, complexipy)
2. `pytest tests/ -v` passes — run from the repo root, not `pytest` bare
3. New code has tests in `tests/`
4. Google-style docstrings on public APIs; comments in English
5. No manual edits to `_vendor/` — update upstream in zerodep, re-vendor via CLI
6. `ruff format` covers both `src/` **and** `tests/`

## Workflow

- **Branch from master**, open a PR, require CI green before merge.
- **Merge strategy: rebase** — keep commits atomic and well-messaged.
- Branch naming: `feature/...`, `fix/...`, `refactor/...`, `test/...`, `docs/...`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Oaklight/ToolRegistry](https://github.com/Oaklight/ToolRegistry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
