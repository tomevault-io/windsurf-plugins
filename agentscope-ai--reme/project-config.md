---
trigger: always_on
description: This file guides coding agents working in the ReMe repository. Keep changes small,
---

# AGENTS.md

This file guides coding agents working in the ReMe repository. Keep changes small,
testable, and consistent with the contracts already expressed by the code.

## Project Principles

ReMe is a local-first, file-native memory system for agents.

- User-owned memory files are the source of truth.
- Indexes, caches, metadata, and generated state must be rebuildable.
- Prefer transparent formats and behavior over hidden state.
- Preserve user control over storage, configuration, and service boundaries.
- Keep concepts focused on project intent; let code and schemas describe implementation.

When a proposed convenience conflicts with these principles, favor data ownership,
recoverability, and predictable behavior.

## Sources of Truth

Use this order when documentation and implementation disagree:

1. Current code and public Pydantic schemas.
2. Tests that describe supported behavior.
3. CLI help and the built-in configuration.
4. Development documentation and historical notes.

Do not copy large implementation descriptions into documentation. Link to the relevant
module or express the stable contract instead. If behavior changes intentionally, update
the code, schema, tests, configuration, and concise documentation together as needed.

## Repository Map

- `reme/reme.py`: CLI entry point and client/server dispatch.
- `reme/application.py`: application assembly, dependency ordering, and lifecycle.
- `reme/components/application_context.py`: application-wide wiring and shared in-memory metadata.
- `reme/components/runtime_context.py`: scratch state shared by steps within one execution.
- `reme/config/default.yaml`: built-in jobs, components, and defaults.
- `reme/schema/`: public and runtime Pydantic contracts.
- `reme/components/`: services, stores, clients, jobs, and component registration.
- `reme/steps/`: executable job steps.
- `tests/unit/`: primary fast validation suite.
- `tests/integration/`: tests that may require real credentials or services.
- `tests/vector/` and `tests/light/`: specialized suites.
- `plugins/reme/`: Claude Code integration.
- `skills/reme_memory/`: skill that communicates with the ReMe service.
- `skills/qwenpaw_memory/`: separate direct-file memory convention; it does not call ReMe.
- `docs/`: pages and assets that support the repository README; not the deployed docs site.

## Development Setup

ReMe requires Python 3.11 or newer.

```bash
pip install -e ".[dev,core]"
```

Before changing behavior, inspect the adjacent implementation, schemas, configuration,
and focused tests. Follow existing patterns unless the task explicitly calls for a new
contract or architecture.

## Change Workflow

1. Identify the narrowest supported contract affected by the request.
2. Read the relevant implementation and tests before editing.
3. Make the smallest coherent change; avoid unrelated cleanup.
4. Update related schemas, defaults, registrations, and imports when required.
5. Add or adjust focused tests for observable behavior.
6. Run proportionate validation and report anything not run.

Component and step discovery depends on registration imports:

- Components use `R.register(...)` in `reme/components/component_registry.py`.
- Component packages must be reachable through `reme/components/__init__.py`.
- Step modules must be reachable through `reme/steps/__init__.py`.

Adding an implementation without its registration import can leave it undiscoverable at
runtime. Treat the implementation, registry entry, and import side effect as one change.

Do not silently change stable CLI flags, configuration keys, workspace layouts, serialized
schemas, or service interfaces. When such a change is required, preserve compatibility
where practical and make the migration explicit.

## Step State Model

Treat every Step as stateless. `BaseJob` stores Step specifications and builds fresh Step
instances for each Job invocation. A Step instance must not use `self` or class variables to
retain mutable runtime state between calls.

Place state according to its lifetime:

- Constructor fields on `self`: immutable Step configuration and resolved dependencies only.
- `self.context` (`RuntimeContext`): request data and intermediate results for one Job
  execution; sequential Steps share this context.
- `self.app_context.metadata`: in-memory state that must be shared across Step or Job
  invocations for the lifetime of the Application.
- Workspace files or a dedicated Component/store: durable state that must survive an
  Application restart.

Use narrow, namespaced keys in `app_context.metadata`, following existing patterns such as
`tool_contexts`. The ApplicationContext is shared, so account for
concurrent access when values are mutable. New Step code must not fall back to `self.kwargs`
or another Step field to emulate shared state when `app_context` is absent; tests of shared
state should construct an `ApplicationContext`. If shared state grows into a stable
service-level contract or needs its own lifecycle, locking, or persistence, promote it to a
typed ApplicationContext field or a dedicated Component instead of expanding an ad hoc
metadata bucket.

Do not use `Response.metadata` as a state store. It is request-scoped output for callers and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentscope-ai/ReMe](https://github.com/agentscope-ai/ReMe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
