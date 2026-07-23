---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dora (AI-Dora, Agentic Dataflow-Oriented Robotic Architecture) is a 100% Rust framework for building real-time robotics and AI applications. It provides 10-17x faster latency than ROS2 via zero-copy shared memory and Apache Arrow data format. Supports Rust, Python (PyO3), C, and C++ nodes.

## Build Commands

```bash
# Build all (Python packages require maturin, exclude them for normal builds)
cargo build --all --exclude dora-node-api-python --exclude dora-operator-api-python --exclude dora-ros2-bridge-python

# Build specific package
cargo build -p dora-cli
cargo build -p dora-daemon

# Check all
cargo check --all

# Test all (excluding Python)
cargo test --all --exclude dora-node-api-python --exclude dora-operator-api-python --exclude dora-ros2-bridge-python

# Test single package
cargo test -p dora-core

# Lint
cargo clippy --all

# Format
cargo fmt --all

# Format check (CI uses this)
cargo fmt --all -- --check

# Run examples
cargo run --example rust-dataflow
cargo run --example benchmark --release

# Install CLI locally
cargo install --path binaries/cli --locked

# Run a dataflow
dora run examples/rust-dataflow/dataflow.yml
dora run examples/python-dataflow/dataflow.yml --uv --stop-after 10s
```

## Workspace Layout

- **Rust edition 2024; MSRV and default workspace package metadata live in `[workspace.package]` in the root `Cargo.toml`.** Most crates inherit the workspace version via `version.workspace = true`, but a few (e.g. `apis/rust/operator/types`, the `examples/error-propagation/*` samples) pin their own version independently.
- Python packages use PyO3 0.28 and are built with **maturin**, not cargo

### Key crates

| Path | Crate | Role |
|------|-------|------|
| `binaries/cli` | dora-cli | CLI binary (`dora` command) - build, run, stop dataflows |
| `binaries/daemon` | dora-daemon | Spawns nodes, manages local shared-memory/TCP communication |
| `binaries/coordinator` | dora-coordinator | Orchestrates distributed multi-daemon deployments |
| `binaries/runtime` | dora-runtime | In-process operator execution runtime |
| `libraries/message` | dora-message | All inter-component message types and protocol definitions |
| `libraries/core` | dora-core | Dataflow descriptor parsing, build utilities, Zenoh config |
| `apis/rust/node` | dora-node-api | Rust API for writing custom nodes |
| `apis/rust/operator` | dora-operator-api | Rust API for writing in-process operators |
| `apis/python/node` | dora-node-api-python | Python node API (PyO3) |
| `libraries/extensions/mavlink2-bridge` | dora-mavlink2-bridge | MAVLink 2 ↔ Apache Arrow conversion (common dialect) |
| `binaries/mavlink2-bridge-node` | dora-mavlink2-bridge-node | Daemon-spawnable MAVLink 2 bridge (TCP/UDP/serial) |

## Architecture

```
CLI  -->  Coordinator  -->  Daemon(s)  -->  Nodes / Operators
              (distributed)    (per machine)    (user code)
```

- **CLI <-> Coordinator**: WebSocket (port 6013) for build/run/stop commands
- **Coordinator <-> Daemon**: WebSocket for node spawning and dataflow lifecycle
- **Daemon <-> Daemon**: Zenoh for distributed cross-machine communication
- **Daemon <-> Node**: Shared memory for messages >4KB (zero-copy), TCP for small messages
- **Data format**: Apache Arrow columnar format throughout (zero serialization overhead)

### Dataflow specification

Dataflows are defined in YAML files. Nodes declare inputs (subscribing to other nodes' outputs) and outputs. Built-in timer nodes: `dora/timer/millis/<N>`, `dora/timer/hz/<N>`.

### Communication patterns

User-facing patterns (see `docs/patterns.md`):
- **Topic**: default pub/sub dataflow
- **Service**: request/reply via `request_id` metadata key; helpers: `send_service_request()`, `send_service_response()`
- **Action**: goal/feedback/result via `goal_id`/`goal_status` metadata keys; supports cancellation

Internal transport:
- **flume** channels: bounded MPSC for internal event routing
- **tokio** async runtime with full features
- **Zenoh**: pub-sub for remote/distributed nodes
- **UHL Clock** (`uhlc`): hybrid logical clock for distributed causality

## Pre-commit Quality Gates (MANDATORY)

**Every commit MUST pass these gates before pushing.** This is a BLOCKING REQUIREMENT — do not skip any step. Remote CI is slower than your laptop and has limited capacity; catching failures locally saves 5-15 minutes per round-trip.

> The bar below is the **Class A** (low-risk) baseline. Behavior
> changes and high-risk subsystem edits have additional requirements
> — see [`docs/agentic-qa-policy.md`](docs/agentic-qa-policy.md)
> (#1634) for the per-class expectations and the PR validation
> summary template.

### Step 1: Run /review

Run the `/review` skill on your changes before committing. This catches structural issues, security problems, and logic errors that tests and clippy miss.

### Step 2: Run /simplify

Run the `/simplify` skill to check for unnecessary complexity, code duplication, or inefficiency in the changed code.

### Step 3: Local CI (fmt + clippy + tests)

```bash
# 1. Format — CI rejects formatting diffs
cargo fmt --all -- --check


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dora-rs/dora](https://github.com/dora-rs/dora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
