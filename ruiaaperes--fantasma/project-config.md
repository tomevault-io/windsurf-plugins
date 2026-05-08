---
trigger: always_on
description: This repository is for building Fantasma, a privacy-first, self-hosted mobile analytics stack. Treat this file as the operating manual for coding agents and contributors.
---

# AGENTS.md

This repository is for building Fantasma, a privacy-first, self-hosted mobile analytics stack. Treat this file as the operating manual for coding agents and contributors.

## Product Ethos

- Fantasma is install-scoped analytics, not person-scoped analytics.
- Simplicity is a product feature. Prefer fewer metrics, fewer dimensions, and fewer knobs.
- The goal is actionable understanding for mobile apps, not exhaustive analytics coverage.
- Privacy claims must be reflected in the data model. Do not add hidden identity or stitching primitives.
- Do not add user-defined event properties to the current event contract or SDK APIs unless the repository explicitly changes direction.
- If event metadata is ever introduced, treat it as event context only and never use it to reconstruct person-level identity.

## Product Rules

- Keep Fantasma simple, API-first, mobile-first, self-hosted, and privacy-first.
- The API is the primary interface.
- Everything is an event.
- Events are immutable.
- Client-side durability is required.
- Backend aggregation is asynchronous.
- Prefer deployability and clarity over infrastructure complexity.

## MVP Defaults

Until the repository says otherwise, assume the following:

- Postgres-only v1
- project-scoped multi-tenant model
- backend + public API + durable iOS SDK are the MVP
- no Redis, Kafka, or extra operational dependencies in v1

## Explicit Non-Goals

Do not add scope beyond the product vision without an explicit decision:

- feature flags
- attribution
- ad tracking
- A/B testing
- session replay
- fingerprinting
- customer data platform behavior

## Backend Rules

- Ingestion happens through `POST /v1/events`.
- Query endpoints live under `/v1/metrics/*`.
- Raw events are stored append-only.
- Workers own aggregate generation.
- Do not perform synchronous aggregation in the ingest path.
- Do not add hidden enrichment or automatic identity stitching.
- Do not add person-level identity primitives to the MVP model.
- Do not accept public session identifiers from clients; backend sessionization is internal.
- Do not infer identity from event payload fields or other client metadata.
- Scope all persisted data by `project_id`.
- DB-backed Rust tests should run fully in Docker through the repository workflow rather than host Postgres or ad hoc host `DATABASE_URL` setup.
- Keep `cargo test --workspace` on the Docker Postgres path limited to tests satisfied by containerized Postgres alone; stack-level checks belong in dedicated smoke workflows.
- When changing metric bucket schemas or replacing metric tables, preserve equivalent bounded read indexes for later-dimension filters and update the EXPLAIN-backed store tests in the same change.

## SDK Rules

- Keep the SDK API minimal and explicit.
- No swizzling.
- No automatic screen tracking.
- No hidden behavior.
- Persist tracked events before upload.
- Do not expose person-scoped identity APIs in the MVP.
- `clear()` rotates local identity without deleting already-queued events.

## Documentation Rules

Documentation is a first-class deliverable.

- Keep the canonical public product/identity/privacy stance in `README.md`.
- `README.md` is a product-facing entry point, not an implementation dump.
- Do not spread the same product-direction explanation across multiple docs with different wording.
- Any public API change must update `schemas/openapi`.
- Any event contract change must update `schemas/events`.
- Any architecture change must update `docs/architecture.md`.
- Any deployment change must update `docs/deployment.md`.
- Public-facing SDK behavior must include usage examples in docs.
- Secondary docs should describe consequences of the canonical README stance:
  - `docs/deployment.md` for operational behavior
  - `docs/architecture.md` for technical design
  - SDK docs for client behavior
  - schemas/OpenAPI for contract text
- Keep `README.md` concise and high-level; technical specifics belong in the derived docs above.
- When public direction changes, update `README.md` first, then align the derived docs.
- Avoid duplicating policy or ethos text across files unless a short local restatement is needed for clarity.
- Work is not complete until documentation is updated.

## Project Memory

Keep a running record of what changed and what is next.

- Update `docs/STATUS.md` when significant work starts.
- Update `docs/STATUS.md` when significant work finishes.
- Record open decisions, active work, and next steps.
- Use commit history for fine-grained change history and `docs/STATUS.md` for current project memory.

## Agent Workflow

Fantasma now expects Codex agents to use the installed `superpowers` skills when they apply. Repository rules in this file still win over any skill defaults.

- Use `writing-plans` before starting multi-step feature or refactor work. Save plans under `docs/superpowers/plans/` unless the task clearly does not need one.
- Use `subagent-driven-development` when executing a plan with separable tasks in the current session.
- When the user reports a bug, do not start by patching code. First write a test that reproduces the bug.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RuiAAPeres/fantasma](https://github.com/RuiAAPeres/fantasma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
