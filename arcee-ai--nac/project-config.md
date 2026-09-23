---
trigger: always_on
description: NAC is a Rust coding-agent harness with a React web client. It supports three
---

# NAC repository guide

NAC is a Rust coding-agent harness with a React web client. It supports three
immutable session behaviors: the established orchestrator/worker topology, a
persistent direct coding agent with traditional child sessions, and a direct
agent that can launch separate managed orchestrator sessions. Managed-host
deployment is additive and opt-in.

Read the nearest nested `AGENTS.md` before changing a subsystem. This root file
defines repository-wide constraints; nested guides add ownership-specific
rules. Durable architectural decisions live in `docs/architecture/`. Historical
work ledgers and local notebooks are evidence, not a substitute for current code
and tests.

## Dependency direction

Dependencies point inward:

```text
React clients
    -> HTTP/OpenAPI/MCP delivery (nac-server)
        -> focused application services and composition ports
            -> nac-core durable harness       nac-managed product context
                -> shared contracts/infrastructure adapters <-
```

- Domain and application contracts must not depend on Axum, React, provider
  wire types, or deployment wiring.
- `nac-managed` does not depend on `nac-server` or the agent harness. The server
  composes its ports with core/project services.
- Provider, filesystem, database, Git/process, sandbox, and HTTP clients are
  adapters around inward contracts; do not move transport types into domain
  APIs.
- Model-visible tool exposure, authorization, hard safety policy, and the
  already-selected execution backend are separate decisions. Approval never
  changes or escapes the backend.
- Prefer private modules and narrow exports. Add a crate only for a real
  dependency boundary, not to increase crate count.

See [dependency boundaries](docs/architecture/0001-dependency-boundaries.md)
and [generated API contract](docs/architecture/0002-generated-api-contract.md).

## Where changes belong

| Change | Owner | Start here |
| --- | --- | --- |
| Model loop, prompts, compaction | `nac-core::agent` | `crates/nac-core/src/agent/` |
| Session admission, recovery, cancellation, settlement | `nac-core::session_service` | `crates/nac-core/src/session_service/AGENTS.md` |
| Durable schema, transcript, relationships, inbox/goals | `nac-core::store` and `sessions` | `crates/nac-core/src/store/AGENTS.md` |
| Permission evaluation and approval | `nac-core::permissions` | `crates/nac-core/src/permissions/AGENTS.md` |
| Native tools, capability composition, invocation | `nac-core::tools` | `crates/nac-core/src/tools/AGENTS.md` |
| Model/runtime/backend construction | `nac-core::runtime` | `crates/nac-core/src/runtime/` |
| Managed secrets, GitHub, clone workflow, readiness | `nac-managed` | `crates/nac-managed/AGENTS.md` |
| Product use cases and HTTP/OpenAPI/MCP delivery | `nac-server` | `crates/nac-server/AGENTS.md` |
| React features, queries, presentation | web client | `crates/nac-server/web/AGENTS.md` |
| Managed image/runtime contract | managed container | `docker/managed/AGENTS.md` |
| Shared command environment and credentials | small inward crates | `crates/nac-contracts/`, `crates/nac-credential-store/` |
| Descendant-aware process supervision | `nac-process` | `crates/nac-process/AGENTS.md` |
| Checked-in model catalog generation | `nac-catalog-gen` | `crates/nac-catalog-gen/AGENTS.md` |

## Product and compatibility invariants

- Persisted session behavior is immutable. Preserve the public values
  `orchestrator`, `direct`, and `direct-with-orchestrator`; omitted legacy values
  still default to `orchestrator`.
- Traditional child sessions and managed orchestrators are distinct durable
  topologies. Do not collapse them into worker roles or a new public taxonomy.
- Preserve prompts, defaults, thread/workset behavior, MCP semantics, API
  shapes, configuration values, and supported migrations unless a deliberate
  product decision changes them.
- Durable inbox, goals, transcripts, relationships, leases, recovery,
  cancellation, and completion delivery must remain restart-safe and
  generation-aware.
- Preserve revision-checked atomic mutation, no-follow path handling, retained
  terminal output, cancellation/process-tree cleanup, rich results, event
  emission, and workspace ownership.
- Permission ordering, hard denials, canonical resource binding, remembered
  grant scope, and headless fail-closed behavior are safety contracts. Tool
  visibility is never authorization.
- Local, SSH, and optional Podman are construction-time backend choices.
  Authorization cannot select a different backend or grant sandbox escape.
- Managed credentials and Exa credentials retain exact-value redaction and
  ownership isolation. Managed features remain absent/inert when not enabled.

Read neighboring tests before changing a safety, durability, schema, recovery,
or concurrency seam. Add characterization coverage with the change.

## Repository map

- `crates/nac-core/` — durable harness, model/runtime loop, tools, permissions,
  sessions, persistence, workspace and execution backends.
- `crates/nac-managed/` — harness-independent managed-host bounded context.
- `crates/nac-server/` — application composition plus HTTP/OpenAPI/MCP delivery
  and the `nac-web` binary.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcee-ai/nac](https://github.com/arcee-ai/nac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
