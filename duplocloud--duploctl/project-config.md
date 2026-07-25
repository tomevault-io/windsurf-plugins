---
trigger: always_on
description: `duploctl` is a **CLI and Python package** for interacting with DuploCloud portals. It provides a command-line interface and extensible Python module for managing DuploCloud resources (Tenants, Services, Infrastructure, etc.) within CI/CD pipelines and interactive workflows.
---

# GitHub Copilot Instructions for duploctl

## Project Overview

`duploctl` is a **CLI and Python package** for interacting with DuploCloud portals. It provides a command-line interface and extensible Python module for managing DuploCloud resources (Tenants, Services, Infrastructure, etc.) within CI/CD pipelines and interactive workflows.

**Key Characteristics:**
- **Dual Interface**: Works as both a CLI (`duploctl <resource> <command>`) and Python module
- **Plugin Architecture**: Uses Python entry points for dynamic resource loading
- **Decorator-Based**: `@Resource` and `@Command` decorators register resources and commands
- **Scope System**: Resources can be portal-scoped or tenant-scoped with automatic injection
- **API Versioning**: Supports v1, v2, and v3 DuploCloud API endpoints with appropriate base classes
- **CRUD Patterns**: V2/V3 base classes provide standardized CRUD operations for applicable resources

## Core Architecture

### Resource Registration System

Resources are registered via **entry points** in `pyproject.toml` and decorated classes:

```toml
[project.entry-points."duplocloud.net"]
tenant = "duplo_resource.tenant:DuploTenant"
service = "duplo_resource.service:DuploService"
infrastructure = "duplo_resource.infrastructure:DuploInfrastructure"
```

Each resource class must:
1. Be decorated with `@Resource(name, scope="portal"|"tenant")`
2. Accept `DuploCtl` in `__init__`
3. Define commands using `@Command()` decorator
4. Be registered in `pyproject.toml` entry points

### Decorator Pattern

#### `@Resource(name: str, scope: str = "portal")`

Registers a class as a resource and optionally injects scope-specific functionality.

**Parameters:**
- `name`: Resource name (used in CLI: `duploctl <name> ...`)
- `scope`: Either `"portal"` (default) or `"tenant"`

**Scope Behaviors:**
- **Portal scope**: Resource operates at portal level, no tenant context required
- **Tenant scope**: Automatically injected with:
  - `tenant` property (lazy-loaded tenant object)
  - `tenant_id` property (lazy-loaded tenant ID)
  - `prefix` property (returns `duploservices-{tenant_name}-`)
  - `prefixed_name(name)` method (prepends tenant prefix)
  - `endpoint()` method (tenant-aware endpoint builder)

#### `@Command(*aliases)`

Registers a method as an executable command. Arguments are automatically parsed from function signature using type hints.

```python
@Command()
def create(self, body: args.BODY, wait: args.WAIT = False) -> dict:
    """Create a resource."""
    # Implementation
```

### Base Classes and API Versions

**When to extend base classes:**
- Only extend `DuploResourceV2` or `DuploResourceV3` if the resource has **CRUD operations** (find, create, update, delete, apply)
- Resources with only custom methods (like `version`, `jit`, `system`, `plan`) should **not extend anything** - just use `@Resource` decorator

**Base Class Hierarchy:**

1. **`DuploResource`** (v1 API - base class)
   - Default `api_version = "v1"`
   - Minimal functionality: `wait()`, `command()`, `__call__()`
   - Use for: Non-CRUD resources like `jit`, `system`, `plan`, `version`

2. **`DuploResourceV2`** (v2 API)
   - Default `api_version = "v2"`
   - Provides: `list()`, `find()`, `apply()` commands
   - Portal endpoint: `endpoint(path)` returns `path`
   - Tenant endpoint: `endpoint(path)` returns `subscriptions/{tenant_id}/{path}`
   - Use for: CRUD resources on v2 API (e.g., `user`, `infrastructure`, `asg`, `hosts`, `lambda`)

3. **`DuploResourceV3`** (v3 API)
   - Default `api_version = "v3"`
   - Provides: `list()`, `find()`, `create()`, `update()`, `delete()`, `apply()` commands
   - Portal endpoint: `endpoint(name, path)` returns `v3/{slug}/{name}/{path}`
   - Tenant endpoint: `endpoint(name, path)` returns `v3/subscriptions/{tenant_id}/{slug}/{name}/{path}`
   - Use for: CRUD resources on v3 API (e.g., `configmap`, `secret`, `batch_*`, `cloudfront`)

**Endpoint Method Behavior:**

The `endpoint()` method is **version-aware** and **scope-aware**:

```python
# V2 Portal: endpoint(path)
self.endpoint("mypath")  # → "mypath"

# V2 Tenant: endpoint(path)
self.endpoint("mypath")  # → "subscriptions/{tenant_id}/mypath"

# V3 Portal: endpoint(name, path)
self.endpoint("myresource", "details")  # → "v3/{slug}/myresource/details"

# V3 Tenant: endpoint(name, path)
self.endpoint("myresource", "details")  # → "v3/subscriptions/{tenant_id}/{slug}/myresource/details"
```

### Tenant Scope Injection Pattern

When `@Resource(scope="tenant")` is used, the decorator **dynamically injects** tenant functionality using a **mixin pattern** (`_inject_tenant_scope`):

1. Wraps `__init__` to add private attributes (`_tenant`, `_tenant_id`)
2. Injects lazy-loading properties using `setattr()`
3. Overrides `endpoint()` method with tenant-aware version
4. No deep inheritance required - clean separation of concerns

**Why this matters:**
- Avoids deep inheritance hierarchies
- Properties are lazy-loaded (no API calls until accessed)
- Single decorator parameter controls behavior
- Works across V2 and V3 APIs consistently


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duplocloud/duploctl](https://github.com/duplocloud/duploctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
