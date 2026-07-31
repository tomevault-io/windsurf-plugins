---
trigger: always_on
description: > **Security-first reverse proxy built to guard the free web.**
---

# Zentinel

> **Security-first reverse proxy built to guard the free web.**

Zentinel is an open-source, high-performance reverse proxy built on Cloudflare's Pingora framework. It emphasizes **predictability**, **transparency**, and **operational simplicity**—infrastructure that lets operators sleep.

## Philosophy (North Star)

Every contribution must align with the [Manifesto](../MANIFESTO.md):

1. **Infrastructure should be calm** — No surprises. Clear limits, predictable timeouts, explainable failure modes.
2. **Security must be explicit** — No magic, no implied policy. Every decision visible and traceable.
3. **The edge is a boundary, not a battleground** — Step in only when necessary, proportionally.
4. **Complexity must be isolated** — Core dataplane stays small. Complex logic lives in external agents.
5. **The web is a commons** — No vendor lock-in, no hidden control planes. Code is readable, forkable, modifiable.
6. **Production correctness beats feature breadth** — Boring reliability over shiny features.

**Before adding anything, ask:**
- Does this introduce ambiguity?
- Can this fail loudly and safely?
- Will this make someone's on-call worse?

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Zentinel Proxy                               │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │                    Pingora Foundation                            ││
│  │  (async I/O, connection pooling, TLS, HTTP/1.1 & HTTP/2)        ││
│  └─────────────────────────────────────────────────────────────────┘│
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐│
│  │ Routing  │ │  Rate    │ │  Cache   │ │ Filters  │ │   Agent    ││
│  │  Engine  │ │ Limiting │ │  Layer   │ │  Chain   │ │  Manager   ││
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └─────┬──────┘│
└────────────────────────────────────────────────────────────┼────────┘
                                                             │
                    ┌────────────────────────────────────────┼────────┐
                    │              External Agents            │        │
                    │  ┌─────────┐ ┌─────────┐ ┌─────────┐  │        │
                    │  │   WAF   │ │  Auth   │ │  Rate   │  │◄───────┘
                    │  │  Agent  │ │  Agent  │ │  Limit  │  │
                    │  └─────────┘ └─────────┘ └─────────┘  │
                    └───────────────────────────────────────────────────┘
```

**Key design choice:** Agents are external processes, not compiled into the proxy. This provides crash isolation, independent deployment, and language flexibility.

---

## Crates

Each crate has its own `docs/` directory with detailed documentation. **When making changes to a crate, update its `docs/` accordingly.**

### Core Crates

#### `zentinel-proxy` (`crates/proxy/`)
Main binary and Pingora integration. Implements HTTP handling, routing, filtering, and upstream communication.
- **Key types:** `ZentinelProxy`, `ProxyApp`
- **Docs:** `crates/proxy/docs/` — architecture, agents, rate-limiting, inference routing, modules

#### `zentinel-config` (`crates/config/`)
KDL configuration parsing, validation, and schema. Handles all configuration file processing.
- **Key types:** `Config`, `RouteConfig`, `UpstreamConfig`, `ListenerConfig`
- **Docs:** `crates/config/docs/` — KDL format, schema reference, validation rules, examples

#### `zentinel-agent-protocol` (`crates/agent-protocol/`)
Agent communication protocol (v2). Handles UDS, gRPC, and reverse connections.
- **Key types:** `AgentPool`, `AgentClientV2`, `Decision`, `ReverseConnectionListener`
- **Docs:** `crates/agent-protocol/docs/` — v2/ protocol specs, API reference, transport options

#### `zentinel-common` (`crates/common/`)
Shared types, utilities, and error handling used across all crates.
- **Key types:** `RequestId`, `Limits`, error types, identifiers
- **Docs:** `crates/common/docs/` — errors, identifiers, limits, observability, patterns

### Supporting Crates

| Crate | Path | Purpose |
|-------|------|---------|
| `playground-wasm` | `crates/playground-wasm/` | WASM module for web playground (config validation) |
| `wasm-runtime` | `crates/wasm-runtime/` | WASM agent runtime using Wasmtime |
| `sim` | `crates/sim/` | Simulation and testing utilities |
| `stack` | `crates/stack/` | Integration test harness |

### Crate Dependencies

```
zentinel-proxy
├── zentinel-config
├── zentinel-agent-protocol
├── zentinel-common
└── pingora (external)

zentinel-config
└── zentinel-common

zentinel-agent-protocol
└── zentinel-common
```

**Dependency rules:**
- `proxy` may depend on all internal crates
- `config` and `agent-protocol` depend only on `common`
- `common` has no internal dependencies

---

## Key Concepts

### Request Lifecycle

1. **Accept** — TCP connection established, TLS handshake (if HTTPS)
2. **Parse** — HTTP request parsed, headers extracted
3. **Route** — Priority-based matching, LRU cached
4. **Filter** — Pre-upstream filters, agent hooks
5. **Upstream** — Load-balanced backend selection, request forwarding
6. **Response** — Response filters, caching, compression
7. **Log** — Access logging, metrics emission


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zentinelproxy/zentinel](https://github.com/zentinelproxy/zentinel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
