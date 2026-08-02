---
trigger: always_on
description: Ursula is a self-hosted, distributed Durable Streams server for replayable, append-only event timelines over plain HTTP/SSE, with Raft durability and S3-backed cold storage. It implements the [Durable Streams Protocol](https://github.com/durable-streams/durable-streams) and is designed for open-source self-hosting, low write latency (sub-50ms P99), plain S3 economics, and quorum-replicated durability.
---

# Ursula Agent Guide

## Project Overview

Ursula is a self-hosted, distributed Durable Streams server for replayable, append-only event timelines over plain HTTP/SSE, with Raft durability and S3-backed cold storage. It implements the [Durable Streams Protocol](https://github.com/durable-streams/durable-streams) and is designed for open-source self-hosting, low write latency (sub-50ms P99), plain S3 economics, and quorum-replicated durability.

The server uses a thread-per-core, multi-Raft architecture: each stream hashes to one Raft group, that group has one replica on each voter node, and the same group ID is owned by a deterministic core on every node. Groups replicate independently with no cross-group transaction path.

- **Version**: 0.3.0 (prototype phase)
- **License**: Apache-2.0
- **Repository**: <https://github.com/tonbo-io/ursula>
- **Homepage**: <https://ursula.tonbo.io>
- **Language**: Rust (Edition 2024)
- **Toolchain**: Nightly (`nightly-2026-06-01`)

## Technology Stack

- **Language**: Rust 2024 Edition, nightly toolchain
- **Async Runtime**: Tokio (multi-thread + single-thread worker pools)
- **HTTP Server**: axum (stateless front door)
- **Raft Consensus**: OpenRaft (patched from databendlabs/openraft)
- **gRPC**: tonic (inter-node Raft RPCs)
- **Serialization**: serde, prost (protobuf)
- **Cold Storage**: OpenDAL (S3, memory backends)
- **Observability**: tracing, OpenTelemetry OTLP
- **Allocator**: mimalloc
- **Simulation**: madsim (deterministic simulation testing)
- **Benchmarking**: criterion
- **Property Testing**: proptest
- **Documentation Site**: Vite + React + MDX (docs/web/)
- **Deployment**: Docker, Helm (Kubernetes)
- **CI/CD**: GitHub Actions

## Repository Layout

### Rust Workspace Crates (`crates/`)

| Crate                  | Description                                                                                                                                   |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `ursula`               | HTTP server, CLI binaries (`ursula`, `ursulagw`), bootstrap wiring, and end-to-end protocol tests.                                            |
| `ursula-runtime`       | Per-core actor runtime: hot ring, cold-tier flush, group engine boundary, cold store integration, and runtime benchmarks.                   |
| `ursula-raft`          | OpenRaft-backed group engine: network, log store, snapshot handling, gRPC Raft plumbing.                                                      |
| `ursula-stream`        | Deterministic stream state machine: bucket/stream commands, responses, snapshots, payload metadata, validation.                               |
| `ursula-shard`         | Bucket/stream routing, core ownership, Raft group placement, shared shard identifiers (`CoreId`, `ShardId`, `RaftGroupId`, `BucketStreamId`). |
| `ursula-proto`         | Shared protobuf schemas (model value types and snapshot frames) reused across crates; replicated commands/responses travel as serde MessagePack. |
| `ursula-config`        | Configuration types and TOML loading for Ursula server.                                                                                       |
| `ursula-observability` | Shared tracing/OpenTelemetry initialization for Ursula binaries.                                                                              |
| `ursula-gateway`       | Gateway binary (`ursulagw`) that routes client HTTP traffic while hiding internal leader redirects.                                           |
| `ursula-bench`         | HTTP/client benchmark harnesses for performance testing.                                                                                      |
| `ursula-ctl`           | Operational CLI (`ursulactl`) for cluster management: drain leaderships, wait for catch-up, rolling restarts.                                 |
| `ursula-sim`           | Deterministic simulation harnesses using madsim for fault injection and invariant checking.                                                   |

### Other Top-Level Directories

- `docs/web/`: Documentation site content (Vite + React + MDX). Deployed to Cloudflare Pages.
- `docs/architecture/`: Architecture design documents (DST, thread-per-core multi-Raft, runtime evaluation).
- `charts/ursula/`: Helm chart for Kubernetes deployment.
- `scripts/`: Repository helper scripts including EC2 test orchestration (`ursula_ec2.py`), chaos testing (`ursula_chaos_agent.py`), and DST audit tools (`scripts/dst/`).
- `target/`: Cargo build output.

## Build System

The project uses Cargo as its build system with a workspace configuration in the root `Cargo.toml`.

### Key Configuration Files

- **`Cargo.toml`**: Workspace manifest with shared dependencies, lints, and profiles.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonbo-io/ursula](https://github.com/tonbo-io/ursula) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
