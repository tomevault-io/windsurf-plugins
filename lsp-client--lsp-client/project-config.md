---
trigger: always_on
description: - Lint & format: `ruff check --fix && ruff format`
---

# AGENTS.md

## Development Commands

- Lint & format: `ruff check --fix && ruff format`
- Type checking: `ty check <dir_or_file>`
- Run tests: `uv run pytest`
- Python: always use `uv`

## Code Style Guidelines

- Types: Full Python 3.12 type annotations required, use `lsp_client.utils.types.lsp_type` for standard LSP types
- Error handling: Must use concrete exception types
- Async: Use async/await, `asyncer.TaskGroup` for concurrency
- Structure: Follow capability-based protocol pattern in `capability/` module

## Project Overview

Python 3.12+ async LSP client library with container-first architecture. Capability-driven design using Protocol mixins for type-safe LSP feature composition.

## Structure

```
lsp-client/
├── src/lsp_client/
│   ├── capability/        # Protocol mixins (request, notification, server_*)
│   ├── clients/           # Language server adapters (Pyright, Rust Analyzer, Deno, etc.)
│   ├── server/            # LocalServer, ContainerServer backends
│   ├── client/            # Core Client base class
│   ├── protocol/          # LSP protocol abstractions
│   ├── jsonrpc/           # JSON-RPC transport layer
│   └── utils/             # Helpers (workspace, config, types)
├── tests/                 # unit/, integration/, e2e/
└── examples/              # Runnable demos (pyrefly.py, pyright_container.py, etc.)
```

## Where to Look

| Task                   | Location                                            | Notes                                              |
| ---------------------- | --------------------------------------------------- | -------------------------------------------------- |
| Add new capability     | `capability/request/` or `capability/notification/` | See `capability/AGENTS.md`                         |
| Add language server    | `clients/<lang>.py`                                 | Follow pattern in `clients/AGENTS.md`              |
| Modify server backends | `server/`                                           | LocalServer (subprocess), ContainerServer (Docker) |
| Change initialization  | `client/abc.py`                                     | Client lifecycle, capability negotiation           |
| LSP type definitions   | `lsp_client.utils.types.lsp_type`                   | Re-export from `lsprotocol`                        |
| Container images       | `server/container.py`                               | Image registry at ghcr.io/lsp-client/\*            |

## Architecture Patterns

### Capability Mixin System

- Capabilities defined as `@runtime_checkable` Protocols
- Client classes inherit mixins: `WithRequestHover`, `WithNotifyDidChange`, etc.
- `capability/build.py` introspects class to build LSP `ClientCapabilities`
- Runtime checks via `check_server_capability()` enforce server support

### Dual Server Strategy

- Every language client provides `LocalServer` (subprocess) and `ContainerServer` (Docker)
- Prioritized fallback: user-provided → local → container
- Container mode: automatic path translation, workspace mounting

### Language Client Pattern

- Inherit from language base: `PythonClientBase`, `RustClientBase`, etc.
- Implement: `create_default_servers()`, `create_default_config()`, `check_server_compatibility()`
- Register in `clients/lang.py` for project-root discovery

## Anti-Patterns (This Project)

- ❌ **DO NOT** use `asyncio.gather()` → Use `asyncer.TaskGroup` instead
- ❌ **DO NOT** skip type annotations → Required for all functions
- ❌ **DO NOT** use bare `except:` → Always specify exception types
- ❌ **DO NOT** modify capability mixins without updating `build.py` registration
- ❌ **DO NOT** create clients without both LocalServer and ContainerServer
- ❌ **DO NOT** use `os.path` → Use `pathlib.Path` (PTH lint rule enforced)
- ❌ **DO NOT** add blanket `type: ignore` → Use specific error codes (PGH003)

## Unique Project Characteristics

### Container-First Design

- Containers are first-class runtime, not an afterthought
- Images auto-updated weekly at ghcr.io/lsp-client/\*
- Path translation transparent to users
- Gated by `settings.enable_container` (default: off as of 0.3.7)

### Protocol-Heavy Architecture

- `typing.Protocol` used extensively for capability contracts
- `@runtime_checkable` enables `isinstance()` checks
- Capability registration via `issubclass()` introspection in `build.py`

### Attrs Over Dataclasses

- Internal models use `attrs` (`@define`, `@frozen`) not stdlib `dataclasses`
- Provides validators, converters, evolve patterns
- Dependency cost justified by feature richness

## Common Gotchas

1. **Capability registration is implicit**: Adding a mixin to a client auto-registers the capability. Missing `build.py` awareness = capability not sent to server.

2. **Container path translation**: Paths inside container differ from host. Use `client.as_uri()` to convert, framework handles rest.

3. **Async context managers required**: Clients MUST be used with `async with`, not manual `__aenter__`/`__aexit__`.

4. **Server compatibility checks run at init**: If `check_server_capability()` fails, client won't start. Override to customize.

5. **Local server auto-install**: LocalServer can install missing servers via `ensure_installed` hook. Requires network, may fail in CI.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lsp-client/lsp-client](https://github.com/lsp-client/lsp-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
