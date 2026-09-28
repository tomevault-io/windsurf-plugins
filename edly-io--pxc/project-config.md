---
trigger: always_on
description: Manages WebSocket subscribers. `subscribe()` creates a `Subscriber(websocket, user_id, permission, course_id, activity_id)`. `publish()` iterates subscribers, filters by context match and permission rank (view=0 < play=1 < edit=2).
---

# PXC

This file provides guidance to AI agents working on the PXC codebase — runtime library, demo app, notebook app, tools, and manifest schema. For creating new sample activities, see the `pxc-build-activity` skill.

## What is PXC

Portable, SandboXed Components — a standard for portable, sandboxed online learning activities. Activities are self-contained packages with:

- **manifest.json** (required) — declares fields, actions, events, capabilities, assets
- **ui.js** (required) — exports `setup(activity)`, renders UI in the browser
- **sandbox.js → sandbox.wasm** (optional) — sandboxed backend logic via WASM Component Model, used for grading and state management

The runtime loads manifests, validates actions/events/fields at runtime, executes WASM sandboxes with capability-based permissions, and routes events between UIs and sandboxes via WebSocket.

## Packaging

The repo is a multi-distribution layout. `pxc` is a PEP 420 namespace package (no `src/pxc/__init__.py`) shared by five independently installable sub-projects, each with its own `pyproject.toml`:

| Distribution   | Path                  | Depends on              |
|----------------|-----------------------|-------------------------|
| `pxc-lib`      | `src/pxc/lib/`        | —                       |
| `pxc-demo`     | `src/pxc/demo/`       | `pxc-lib`               |
| `pxc-notebook` | `src/pxc/notebook/`   | `pxc-lib`               |
| `pxc-lti`      | `src/pxc/lti/`        | `pxc-lib`               |
| `pxc-xblock`   | `src/pxc/xblock/`     | `pxc-lib`, `xblock`, `Django` |

The root `pyproject.toml` ships no runtime code — it holds the shared `black` / `mypy` / `pylint` configuration and the `[project.optional-dependencies] dev` group. Use `make install-dev` to install all five sub-projects in editable mode together with the dev tools.

## Codebase layout

```
src/pxc/
  lib/                    Core runtime library (Python) — distribution: pxc-lib
    pyproject.toml
    runtime.py            ActivityRuntime — central orchestrator
    sandbox.py            WASM Component Model executor (wasmtime)
    fields.py             Field type/scope validation (FieldChecker)
    actions.py            Client-to-server action validation (ActionChecker)
    events.py             Server-to-client event validation (EventChecker)
    capabilities.py       Capability enforcement (CapabilityChecker)
    event_bus.py          In-memory pub/sub for WebSocket events (EventBus)
    field_store.py        Abstract FieldStore + MemoryKVStore
    file_storage.py       Abstract FileStorage + LocalFileStorage + MemoryFileStorage
    permission.py         Permission enum (view, play, edit)
    manifest_types.py     Auto-generated Pydantic models from schema (DO NOT EDIT)
    sandbox/
      manifest.schema.json  JSON Schema for activity manifests
      pxc.wit              Canonical WIT: types + state/grading/http/storage/analytics interfaces
    static/js/pxc.js      <pxc-activity> web component (shared across apps)
    tools/
      validate_manifest.py  Manifest validation script
      cache_component.py    Pre-compile WASM components for faster startup
    tests/
      runtime/            Runtime integration tests
      samples/            Per-sample activity tests
        conftest.py       make_runtime() helper — creates ActivityRuntime with MemoryKVStore + MemoryFileStorage
      test_fields.py, test_actions.py, test_events.py, test_capabilities.py, etc.

  demo/                   Minimal demo FastAPI app (port 9752) — distribution: pxc-demo
    pyproject.toml
    app.py                Routes, WebSocket, asset serving
    kv.py                 KVStore (MemoryKVStore subclass with JSON file persistence)
    templates/            Jinja2 templates
    tests/

  notebook/               PXC notebook app (port 9753) — distribution: pxc-notebook
    pyproject.toml
    app.py                FastAPI server — REST API + WebSocket + activity execution
    models.py             SQLModel: Course, Page, PageActivity
    db.py                 SQLite database setup + engine
    field_store.py        SQLiteFieldStore (FieldEntry, FieldLogEntry, FieldLogSeq tables)
    llms.py               LLM integration
    migrations/           Alembic migrations
    frontend/             Next.js static app (TypeScript, React)
    tests/

  lti/                    LTI 1.3 tool provider (port 9754) — distribution: pxc-lti
    pyproject.toml
    app.py                FastAPI server — OIDC, deep linking, resource link launches
    config.py             Env-driven configuration (LTI_BASE_URL, etc.)
    integration.py        Bridge between LTI launches and ActivityRuntime
    core/
      routes.py           LTI endpoints (login, launch, jwks, deep linking)
      oidc.py             OIDC authentication flow
      launch.py           JWT validation + launch handling
      deep_linking.py     Deep linking message construction
      keys.py             RSA keypair management / JWKS
      models.py           Platform registration models
      db.py               Platform storage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edly-io/pxc](https://github.com/edly-io/pxc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
