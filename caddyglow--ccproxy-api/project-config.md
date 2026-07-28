---
trigger: always_on
description: Our primary goal is to build a robust, maintainable, scalable, and secure CCProxy API Server. These conventions are rooted in the following principles:
---

# CCProxy Coding Conventions

## 1. Guiding Principles

Our primary goal is to build a robust, maintainable, scalable, and secure CCProxy API Server. These conventions are rooted in the following principles:

* **Clarity over Cleverness:** Code should be easy to read and understand
* **Explicit over Implicit:** Be clear about intentions and dependencies
* **Consistency:** Follow established patterns within the project
* **Single Responsibility Principle:** Each module, class, or function should have one clear purpose
* **Loose Coupling, High Cohesion:** Modules should be independent but related components within a module should be grouped
* **Testability:** Write code that is inherently easy to unit and integration test
* **Pythonic:** Embrace PEP 8 and the Zen of Python (`import this`)

## 2. General Python Conventions

* **PEP 8 Compliance:** Adhere strictly to PEP 8
  * Use `ruff format` for auto-formatting to ensure consistent style
  * Line length limit is **88 characters** (ruff's default)
* **Python Version:** Target **Python 3.11+**. Utilize modern features like union types (`X | Y`)
* **No Mutable Default Arguments:** Avoid using mutable objects as default arguments
  * **Bad:** `def foo(items=[])`
  * **Good:** `def foo(items: list | None = None): if items is None: items = []`

## 3. Naming Conventions

* **Packages/Directories:** `snake_case` (e.g., `api`, `claude_sdk`, `auth`)
* **Modules:** `snake_case` (e.g., `manager.py`, `client.py`)
* **Classes:** `CamelCase` (e.g., `OpenAIAdapter`, `ServiceContainer`)
  * **Abstract Base Classes:** Suffix with `ABC` or `Protocol`
  * **Pydantic Models:** `CamelCase` (e.g., `MessageCreateParams`)
* **Functions/Methods/Variables:** `snake_case` (e.g., `handle_request`, `get_access_token`)
* **Constants:** `UPPER_SNAKE_CASE` (e.g., `DEFAULT_PORT`, `API_VERSION`)
* **Private Members:** `_single_leading_underscore` for internal use

## 4. Imports

* **Ordering:** Standard library → Third-party → First-party → Relative
* **Absolute Imports Preferred:** Use absolute imports for modules within `ccproxy`
  * **Good:** `from ccproxy.auth.manager import AuthManager`
* **Relative Imports:** Use for modules within the same package
  * **Good (inside `plugins/claude_api/`):** `from .models import ClaudeModel`
* **`__all__` in `__init__.py`:** Define to explicitly expose public API

## 5. Typing

Type hints are mandatory for clarity and maintainability:

* **All Function Signatures:** Type-hint all parameters and return values
* **Class Attributes:** Use type hints, especially for Pydantic models
* **Union Types:** Use `Type | None` for optional values (Python 3.11+)
* **Type Aliases:** Define in `core/types.py` for complex types

## 6. Plugin Architecture

### Plugin Structure
Each plugin must follow the delegation pattern:

```python
plugins/
├── plugin_name/
│   ├── __init__.py
│   ├── adapter.py          # Main plugin interface
│   ├── plugin.py           # Plugin declaration
│   ├── transformers/       # Request/response transformation
│   │   ├── request.py
│   │   └── response.py
│   ├── detection_service.py # Provider capability detection
│   ├── format_adapter.py   # Protocol conversion (if needed)
│   └── auth/               # Authentication (if needed)
│       └── manager.py
```

### Delegation Pattern
Adapters integrate via explicit dependencies (HTTP client, auth manager, transformers) and the application request lifecycle:

```python
class ProviderAdapter(BaseAdapter):
    async def handle_request(self, request, endpoint, method):
        # resolve endpoint/handler config, then execute with injected services
        target_url, needs_conversion = await self._resolve_endpoint(endpoint)
        cfg = await self._create_handler_config(needs_conversion)
        return await self._execute_request(
            method=method,
            target_url=target_url,
            body=await request.body(),
            auth_headers={},
            access_token=None,
            request_headers=dict(request.headers),
            handler_config=cfg,
            endpoint=endpoint,
            needs_conversion=needs_conversion,
            request_context=RequestContext.get_current(),
        )
```

### Format Adapters
- Declarative only: plugins declare adapters in `PluginManifest.format_adapters` with an optional `priority` (lower wins).
- Registration: core pre-registers a few built-in adapters; plugin-declared adapters are registered from manifests during startup.
- Conflicts: resolved by priority during registry finalization; the winning adapter is selected automatically.
- Manual setup: runtime `_setup_format_registry()` is a no-op; avoid calling `registry.register()` from plugins (tests may do so explicitly).
- No global flags: feature flags for adapter selection were removed; manifest-based behavior is always enabled.

## 7. Error Handling

* **Custom Exceptions:** Inherit from `ccproxy.core.errors.CCProxyError`
* **Catch Specific Exceptions:** Never use bare `except:`
* **Chain Exceptions:** Use `raise NewError(...) from original`
* **FastAPI HTTPException:** Use in routes with appropriate status codes

## 8. Asynchronous Programming


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CaddyGlow/ccproxy-api](https://github.com/CaddyGlow/ccproxy-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
