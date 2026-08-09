---
trigger: always_on
description: transforms — round-trip, ordering, idempotency properties.
---

# CLAUDE.md — Conduit

Operational context for Claude Code sessions working on the ConduitIO org. Read this before
touching anything. This file is what a session needs to **act**. The strategy behind the
direction — competition, positioning, commercial line, SDK rationale — lives in `STRATEGY.md`
(gitignored, internal); read it only when doing strategy or positioning work.

## What we're doing

Reviving Conduit (<https://github.com/ConduitIO/conduit>) as the de facto Kafka Connect replacement:
broker-neutral, any-language plugins via gRPC + WASM, embeddable, agent-legible, with a
first-class Kafka Connect migration path and a lightweight state layer. The public plan is in
`ROADMAP.md` — check the current phase at the start of every session and ask which phase we're
executing if it's ambiguous.

## Project reality (read once, keep in mind)

- The project went dormant (~15 months, last v0.14.x nightly 2025-03, restarted 2026-06). The
  v0.15.0 nightly train is **already running**. Phase 0's release task is "cut v0.15.0 stable,"
  not "start v0.15.0."
- **Maintainer reality: solo (DeVaris) + Claude.** The PR process below is written for that,
  not for a large org. Where a gate assumes a "second human maintainer," the second human is
  DeVaris. Gates that require a team switch on by phase — see _Process maturity_ below. Do not
  pretend a review bar is met when it structurally can't be; say what was actually verified.

## Repo conventions (match what exists — do not invent parallel ones)

- **ADRs** live in `docs/architecture-decision-records/`, filename `YYYYMMDD-slug.md`, sections
  `# Title` → `## Summary` → `## Context` → `## Decision` → `## Consequences` → `## Related`.
  Immutable once merged (supersede with a new ADR; never edit). **Not** `docs/adr/`, **not**
  numbered `ADR-001`.
- **Design docs** live in `docs/design-documents/`, same `YYYYMMDD-slug.md` naming. **Not**
  `docs/design/`.
- **Contributing** guide is `CONTRIBUTING.md` (exists) — extend it, don't replace it.
- Existing package layout is documented in `docs/package_structure.md`; code guidelines in
  `docs/code_guidelines.md`. Read those before proposing structural changes.

## Repo map

| Repo | Purpose |
| --- | --- |
| `ConduitIO/conduit` | Core engine: pipeline orchestration, plugin runtime, HTTP/gRPC API, built-in UI |
| `ConduitIO/conduit-connector-protocol` | gRPC protocol between Conduit and connector plugins. **Breaking-change territory — flag loudly, version carefully** |
| `ConduitIO/conduit-connector-sdk` | Go SDK for building connectors (source + destination), acceptance test harness |
| `ConduitIO/conduit-processor-sdk` | Go SDK for processors; standalone processors compile to WASM (wazero runtime) |
| `ConduitIO/conduit-commons` | Shared types: records, schema, config |
| `ConduitIO/benchi` | Benchmarking framework — use for all performance claims |
| `ConduitIO/conduit-connector-*` | Individual connectors (Postgres, Kafka, S3, generator, file, log) |

New surfaces come online across Phases 1–3: connector/template registry, MCP server,
`libconduit` + language bindings, state layer (in-engine), fleet console, K8s operator,
Terraform provider.

## Data integrity invariants (non-negotiable)

Bugs here corrupt people's pipelines. Corruption-class bugs are sev-0: drop everything, fix,
postmortem, regression test.

1. **Never acknowledge a record upstream before it is durably handled downstream.** Ack
   propagation is end-to-end; no intermediate component may ack early for throughput. Any
   batching/buffering change must prove ack correctness under crash.
2. **Positions/offsets are monotonic and crash-safe.** A restart must never skip records
   (at-least-once) or corrupt position state. Position serialization changes require a versioned
   migration path and an upgrade test.
3. **At-least-once is the floor.** Any path that could drop a record without delivering it or
   routing it to a DLQ is a data-loss bug — including error, shutdown, and rebalance paths.
4. **Ordering guarantees are per-source-partition and documented.** Changes that could reorder
   records within a partition key require a design doc and explicit sign-off.
5. **State and checkpoint writes are atomic.** Torn writes on crash must be impossible
   (write-ahead + rename, or the store's transactional API). Every state feature ships with a
   kill-mid-write recovery test.
6. **Schema handling never silently mangles data.** Unknown fields, type mismatches, and drift
   follow the configured policy (halt/DLQ/evolve) — never silent coercion or truncation.
7. **Shutdown is graceful by default.** SIGTERM drains in-flight records and checkpoints before
   exit. `kill -9` at any instant must be recoverable without loss — and we test exactly that.

Where code upholds one of these, say so at the enforcement site:
`// Invariant 1: ack only after destination confirms durable write`.

## Architecture & design discipline (staff/principal bar)

We're building infrastructure people bet their data on. Design like it.

- **Design doc before code for anything non-trivial.** Non-trivial = touches the data path,
  changes a public contract (protocol, config schema, CLI, error codes, state format), adds a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ConduitIO/conduit](https://github.com/ConduitIO/conduit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
