---
trigger: always_on
description: Aranya is a zero-trust security framework for decentralized applications.
---

# AGENTS.md

## What is Aranya

Aranya is a zero-trust security framework for decentralized applications.

- Devices form a **team**, sharing a CRDT-backed DAG (directed acyclic graph).
- Every mutation is a **command**. Commands are synced to all devices and each device independently enforces the policy before accepting a command into its local graph (endpoint enforcement).
- The policy defines RBAC roles (owner / admin / operator / member plus application-defined custom roles), device lifecycle, label-based access control, and AFC (Aranya Fast Channels) permissions.
- There is no central authority -- every device stores a copy of the graph containing all commands it has generated or synced so far, and independently enforces the same policy. When all devices sync without generating new commands they converge to the same full graph, but at any point in time a device may have only a partial view. Devices converge via CRDT sync.
- Separation of duties is enforced: no device can self-promote or bypass the role hierarchy.
- The system has two planes:
  - **Control plane (on-graph):** commands stored in the DAG, broadcast to all devices via sync. Used for access-control operations (add device, assign role, manage labels). Low throughput (~100s msgs/sec), high resilience.
  - **Data plane (off-graph/AFC):** encrypted point-to-point channels between devices, governed by labels in policy. High throughput, low latency. Keys are negotiated on-graph but data flows off-graph.

For full documentation see <https://github.com/aranya-project/aranya-docs/> (includes guides, architecture deep-dives, and API reference; clone the repo for offline access).

## How It Fits Together

```
Application -> Client lib (tarpc RPC) -> Daemon -> Policy engine / Keystore / Sync
```

- One daemon per device; it owns crypto, policy enforcement, and graph sync.
- The daemon runs background sync continuously, exchanging commands with peers over QUIC so each device's graph stays up to date.
- Client libraries (Rust + C bindings) issue RPCs to the local daemon.
- Sync uses QUIC transport; CRDT semantics ensure all devices converge.
- Keys are managed by the daemon's keystore (`crates/aranya-daemon/src/keystore.rs`).

## Crate Map

| Crate | Purpose |
|---|---|
| `aranya-daemon` | Long-running daemon: policy, crypto, sync, device/team state |
| `aranya-client` | Rust client library (tarpc RPC to daemon) |
| `aranya-client-capi` | C bindings (cdylib) for aranya-client |
| `aranya-daemon-api` | Shared RPC/service definitions |
| `aranya-keygen` | CLI for generating device keys |
| `aranya-certgen` | CLI for generating root CA / signed P-256 certs |
| `aranya-util` | Shared utilities (QUIC helpers, async) |
| `aranya-metrics` | Observability: Datadog/Prometheus/TCP exporters |

Examples live in `examples/rust/` and `examples/c/`.

## Policy

The policy is the heart of Aranya. It lives at `crates/aranya-daemon/src/policy.md` as a literate markdown file (>100 KB). Only code inside ` ```policy ``` ` fences is compiled (via `policy-ifgen` at build time). The surrounding prose documents invariants -- always update prose when changing code.

Key sections in the policy: Devices & Identity, Roles & Permissions, Teams, AFC/Labels.

### Vocabulary

- **Action:** The application's entry point into the policy. Actions run **once, on the authoring device only**. They perform checks, prepare data, and `publish` zero or more commands. Actions execute atomically -- if any published command fails, the entire action is rolled back. Because actions only run locally, they are the right place for sensitive operations (e.g. encryption, secret handling) and for logic that should not be visible to other nodes. Iteration over facts (`map`) is only allowed inside actions.
- **Command:** The fundamental unit of the DAG. A command defines structured data (`fields` block), serialization (`seal`/`open` blocks), and validation + side-effects (`policy` and `recall` blocks). **Command `policy` blocks are executed on every device that receives the command** -- this is how endpoint enforcement works. This means command fields are visible to all nodes, so never put plaintext secrets in command fields. Any sensitive data must be encrypted before being placed in command fields (done in the action or in Rust code calling the action).
- **Effect:** A struct emitted from a `finish` block to communicate outcomes back to the application. Along with actions, effects form the only public interface of a policy -- everything else is an implementation detail.
- **Fact:** A key-value pair in the FactDB. Facts are the persistent state that policy reads (via `query`) and writes (via `create`/`update`/`delete` in `finish` blocks). Facts can be declared `immutable` (create/delete only, no update).
- **FFI:** Foreign function interface modules (`use crypto`, `use envelope`, `use device`, etc.) that bring external logic (cryptography, envelope inspection, device identity) into policy. FFI functions must be side-effect-free since policy may be evaluated more than once.

### Execution model

On the **authoring device** (runs once):
1. Application calls an **action** via the client library.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aranya-project/aranya](https://github.com/aranya-project/aranya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
