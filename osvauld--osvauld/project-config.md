---
trigger: always_on
description: Osvauld is an offline-first P2P collaborative platform built on QUIC (iroh), Loro CRDTs, and UCAN capability-based authorization. Two binaries: `sthalam` (desktop app with Slint UI) and `kunki` (always-on headless node).
---

# Osvauld

Osvauld is an offline-first P2P collaborative platform built on QUIC (iroh), Loro CRDTs, and UCAN capability-based authorization. Two binaries: `sthalam` (desktop app with Slint UI) and `kunki` (always-on headless node).

## Offline-First Principle

**CRITICAL**: Osvauld is offline-first. ALL features must work without network connectivity.

- Data is written locally to CRDTs (Loro) first, synced asynchronously when peers are connected
- There are **NO request-response patterns** except `GetShareableLinkRequest -> GetShareableLinkResponse`
- Apps must work fully without network -- the UI is never blocked on a network call
- Sync is convergent and eventual, not transactional
- New protocol message types must be fire-and-forget or multi-step async, never request-response
- Permits are issued once and cached locally -- no online lookups for authorization

When someone proposes a feature that seems to need request-response, the correct approach is: write locally, sync asynchronously, handle convergence via CRDT merge semantics.

## Mandatory Subagent Delegation

**Unless the user explicitly says "do this yourself" or "don't use subagents"**, you MUST delegate to the appropriate subagent via the Task tool. Direct handling is only acceptable for:

1. Trivially small tasks (a one-line fix where spinning up a subagent is overkill)
2. Cross-cutting work that genuinely doesn't map to any single subagent
3. The user explicitly requests direct handling

### Delegation Map

| Task keywords | Delegate to |
|---------------|-------------|
| scribe, CRDT, Loro, layer sync, LayerUnit, state vectors, observer | `@scribe` |
| handshake, sync protocol, SyncOffer, publish, viewer flow, PeerActor | `@peer-actor` |
| coordinator, peer lifecycle, connection routing, actor spawning | `@coordinator` |
| storage, redb, butler, services API, auth, page encryption | `@butler` |
| slint, UI, VecModel, AppAPI, page shell, hot-reload, shell callbacks | `@slint` |
| scribe:bind, scribe:rebind, declarative binding, surgical update, transform | `@scribe-api` |
| lua, runtime, bindings, scheduler, event bus, mlua, on_init | `@lua` |
| permit template, layer patterns, issue_on, dynamic schema, delegation chain | `@permit-template` |
| gurkha, permit parsing, decision engine, authorization, SyncContext | `@gurkha` |
| integration test, Scenario builder, MockConnection, Tracer | `@integration-test` |
| e2e test, python test, AppTestScenario, ControlClient | `@e2e-test` |
| debug, tracing, JSONL, sync divergence, flamegraph, heaptrack | `@debug` |
| documentation, docs update, doc accuracy | `@docs` |
| architecture, crate boundaries, design review, dependency analysis | `@architect` |
| architecture, crate boundaries, design review, dependency analysis | `@architect` |

For multi-domain tasks, coordinate multiple subagents. Example: "Add a new dynamic layer type" involves `@permit-template` (design the schema), `@scribe` (implement layer handling), `@lua` (add Lua bindings), and `@integration-test` (write tests).

## Crate Boundaries

| Crate | Purpose | Dependencies |
|-------|---------|--------------|
| `domains` | Shared domain types (Layer, Page, Space, Identity) | - |
| `herald` | Identity, encryption, signing (Ed25519/X25519) | - |
| `gurkha` | Permit parse + validate only (UCAN) | herald (for crypto) |
| `transport` | QUIC connections via iroh, message framing | iroh |
| `scribe` | CRDT document actor (ractor + Loro) | domains, gurkha, herald |
| `butler` | Storage (redb), services facade API | domains, scribe, herald, gurkha |
| `courier` | P2P orchestration, handshakes, sync protocol | transport, domains, gurkha, butler, herald |
| `lua_runtime` | Lua VM (mlua), app bindings | domains, scribe, butler |
| `renderer_slint` | Slint UI interpreter runtime | lua_runtime, butler, scribe |
| `renderer_raylib` | Raylib graphics renderer | lua_runtime, butler, scribe |
| `sthalam_shell` | Desktop app compiled Slint shell | butler, courier, herald |
| `sthalam` | Desktop app binary (orchestrates everything) | all above |
| `kunki` | Always-on node runtime | butler, courier, herald, transport, gurkha, lua_runtime |
| `control_server` | Unix socket JSON-RPC API | butler (optional) |
| `logging_utils` | Tracing infrastructure | - |

**Rules**:
- Context doesn't leak across crate boundaries
- Butler has a services-only API (no direct store access from outside)
- Courier communicates with apps via async channels
- Each crate wraps its dependency errors -- errors never leak across boundaries
- Scribe is storage-agnostic via 4 trait boundaries (LayerStorage, PeerVectorStorage, PeerResolver, PermitIssuer)

## Code Policies

### Logging

| Level | Use Case | Production |
|-------|----------|------------|
| `trace!` | Wire-level: bytes, serialization | Filtered out |
| `debug!` | Implementation details: cache hits | Filtered out |
| `info!` | Business events: connections, sync | Visible |
| `warn!` | Recoverable issues: retry, fallback | Visible |
| `error!` | Failures requiring attention | Visible |

- Use `#[instrument]` on ALL async functions (don't skip params)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osvauld/osvauld](https://github.com/osvauld/osvauld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
