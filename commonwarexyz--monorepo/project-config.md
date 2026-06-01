---
trigger: always_on
description: This file provides guidance to agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Repository Overview

Commonware is a Rust library providing high-performance, production-ready distributed systems primitives for adversarial environments. It's organized as a Cargo workspace with many primitives that build on each other (sharing testing infrastructure, types, traits, etc.).

## Essential Commands

### Quick Reference

```bash
# Build entire workspace
cargo build --workspace --all-targets

# Test specific crate
just test -p commonware-cryptography

# Test single function
just test -p commonware-consensus test_name

# Run benchmarks
cargo bench -p commonware-cryptography

# Update all crate versions
./scripts/bump_versions.sh <new_version>
```

_For linting, formatting, fuzzing, and other CI-related commands, see the [CI/CD Pipeline](#cicd-pipeline) section below._

## Architecture

### Core Primitives

- **broadcast**: Disseminate data over a wide-area network.
- **codec**: Serialize structured data.
- **coding**: Encode data to enable recovery from a subset of fragments.
- **collector**: Collect responses to committable requests.
- **conformance**: Automatically assert the stability of encoding and mechanisms over time.
- **consensus**: Order opaque messages in a Byzantine environment.
- **cryptography**: Generate keys, sign arbitrary messages, and deterministically verify signatures.
- **deployer**: Deploy infrastructure across cloud providers.
- **p2p**: Communicate with authenticated peers over encrypted connections.
- **resolver**: Resolve data identified by a fixed-length key.
- **runtime**: Execute asynchronous tasks with a configurable scheduler.
- **storage**: Persist and retrieve data from an abstract store.
- **stream**: Exchange messages over arbitrary transport.

Supporting crates include **formatting**, **invariants**, **macros**, **math**, **parallel**, and **utils**.

_All workspace crates can be found in the [Cargo.toml](Cargo.toml) file (anything with a `commonware-` prefix)._

### Examples

- **alto** (https://github.com/commonwarexyz/alto): A minimal (and wicked fast) blockchain built with the Commonware Library.
- **bridge** (`examples/bridge`): Send succinct consensus certificates between two networks.
- **chat** (`examples/chat`): Send encrypted messages to a group of friends.
- **estimator** (`examples/estimator`): Simulate mechanism performance under realistic network conditions.
- **flood** (`examples/flood`): Spam peers deployed to AWS EC2 with random messages.
- **log** (`examples/log`): Commit to a secret log and agree to its hash.
- **reshare** (`examples/reshare`): Reshare threshold cryptographic material across participants.
- **sync** (`examples/sync`): Synchronize state between a server and client.

### Key Design Principles

1. **The Simpler The Better**: Code should look obviously correct and contain the minimum features necessary to achieve a goal.
2. **Test Everything**: All code should be designed for deterministic and comprehensive testing. We employ an abstract runtime (`runtime/src/deterministic.rs`) commonly in the repository to drive tests.
3. **Performance Sensitive**: All primitives are optimized for high throughput/low latency.
4. **Adversarial Safety**: All primitives are designed to operate robustly in adversarial environments.
5. **Abstract Runtime**: Protocol primitives should stay runtime-agnostic. Do not introduce direct `tokio` usage unless the crate already owns a runtime integration or command-line runtime path (for example `runtime`, `deployer`, runtime utilities, benches, or tests). When requiring some `runtime`, use the provided traits in `runtime/src/lib.rs`.
6. **Always Commit Complete Code**: When implementing code and writing tests, always implement complete functionality. If there is a large task, implement the simplest possible solution that works and then incrementally improve it.
7. **Own Core Mechanisms**: If a primitive relies heavily on some core mechanism/algorithm, we should implement it rather than relying on external crates.

### Stability Levels

All public primitives are annotated with stability levels that constrain what changes are permitted:

| Level        | Index | Description                                                                              |
|--------------|-------|------------------------------------------------------------------------------------------|
| **ALPHA**    | 0     | Breaking changes expected. No migration path provided.                                   |
| **BETA**     | 1     | Wire and storage formats stable. Breaking changes include a migration path.              |
| **GAMMA**    | 2     | API stable. Extensively tested and fuzzed.                                               |
| **DELTA**    | 3     | Battle-tested. Bug bounty eligible.                                                      |
| **EPSILON**  | 4     | Feature-frozen. Only bug fixes and performance improvements accepted.                    |

**When modifying code at ALPHA**: Breaking changes to APIs, wire formats, and storage formats are permitted without a migration path. Do not add backwards-compatibility shims or preserve old behavior.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [commonwarexyz/monorepo](https://github.com/commonwarexyz/monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
