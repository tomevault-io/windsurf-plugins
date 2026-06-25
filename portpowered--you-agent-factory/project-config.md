---
trigger: always_on
description: This repository is `you-agent-factory`: a Go, OpenAPI, and React system for
---

# Repository Agent Instructions

This repository is `you-agent-factory`: a Go, OpenAPI, and React system for
scheduling and orchestrating many AI workers concurrently through the `you` CLI,
backend runtime, and dashboard.

The product model is a factory: users define work types, work states,
workstations, workers, guards, resources, and orchestration policy. The runtime
turns submitted work and worker output into ordered events, then derives live
and historical world state from that event stream.

## Current Architecture

- The backend core is an event-first factory runtime. Workers and agents do not
  mutate canonical state directly; they emit outputs that re-enter the runtime
  as events.
- `FactoryService` coordinates APIs, CLI calls, session registries, persistence,
  runtime construction, and model/runtime dependencies. Per-session runtime
  state belongs to the session runtime, not the service coordinator.
- Public terminology is defined in
  `docs/architecture/data-model.md`: `Factory`, `Factory Session`, `Current
  Factory`, `Work`, `Work Request`, and `Provider Session`.
- The customer-facing model hides the internal Petri-net implementation.
  Internal packages can use tokens, places, transitions, markings, and guards;
  public docs and API surfaces should prefer customer-facing vocabulary.
- Factory events and projections are canonical for replay, dashboard state,
  session lifecycle, dispatch lifecycle, work payload lineage, and historical
  inspection.
- The frontend consumes generated OpenAPI types and event streams, derives
  client-side dashboard/editor projections, and sends user actions back through
  API requests. The backend remains authoritative for runtime state.
- OpenAPI is authored in component fragments under `api/components/` and bundled
  into `api/openapi.yaml`; generated Go and TypeScript clients are derived from
  that contract.

Read these architecture notes when the work touches their area:

- `docs/architecture/architecture.md` for backend loop, service/session
  boundaries, event stream, frontend composition, and graph-editor state flow.
- `docs/architecture/data-model.md` for public resource vocabulary and the
  customer/internal data-model split.
- `docs/architecture/invocation-contract.md` for `you run --factory`,
  invocation input resolution, return policy, and CLI/API equivalence.
- `docs/architecture/managed-model-runtime.md` for managed runtime identity,
  readiness, lifecycle, pull/install outcomes, and invocation gating.
- `docs/internal/processes/api-relevant-files.md` when changing public REST
  contract, generated OpenAPI types, factory-session surfaces, events, managed
  runtime surfaces, workflow preview/session APIs, or dashboard API consumers.
- `docs/internal/processes/invocation-relevant-files.md` when changing
  invocation input, return-policy, or primary-result behavior.

## Standards

Start with `docs/internal/standards/STANDARDS.md`, then read the relevant
standard before changing code:

- `docs/internal/standards/code/code-review-standards.md` for reviews and PR
  quality gates.
- `docs/internal/standards/code/general-backend-standards.md` for Go backend,
  state management, architecture, linting, testing, CI, and complexity.
- `docs/internal/standards/code/general-website-standards.md` for React,
  accessibility, responsive design, styling, state, performance, and tests.
- `docs/internal/standards/code/planning-standards.md` for PRDs, work stories,
  acceptance criteria, and implementation plans.

Treat files under `docs/internal/standards/` as normative. Development notes and
process maps can provide examples and file inventories, but they do not override
the standards.

## Repository Map

- `api/` contains the authored OpenAPI entrypoint, component fragments,
  reusable parameters/responses/schemas, and generated bundled contract.
- `cmd/` contains Go command entrypoints and maintenance tools, including the
  main `you` binary under `cmd/factory/`.
- `docs/` contains public docs, architecture notes, comparison docs, examples,
  and internal maintainer standards/process docs.
- `docs/reference/` is the canonical packaged `you docs <topic>` markdown.
  Edit reference topics there, not in generated or embedded copies.
- `examples/` contains example factory directories.
- `factory/` contains this repository's checked-in factory scaffold and
  factory-local docs.
- `pkg/api/` contains HTTP handlers, generated server contracts, API boundary
  tests, OpenAPI contract tests, server tests, and API test data.
- `pkg/apisurface/` contains mapping and normalization at the public API
  boundary. Prefer this layer for transport-independent request/response
  shaping.
- `pkg/cli/` contains CLI commands and CLI adapters for shared service logic.
- `pkg/config/` contains factory config loading, persistence, mapping,
  validation entrypoints, built-in factory layout, and runtime config
  projections.
- `pkg/factory/` contains the core runtime engine, event history, projections,
  requests, validation, scheduling, subsystems, runtime support, and workstation
  config plumbing.
- `pkg/factorysessions/` and `pkg/factorysessionexecution/` contain
  session-level projection/read models and durable/live session execution
  contracts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [portpowered/you-agent-factory](https://github.com/portpowered/you-agent-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
