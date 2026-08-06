---
trigger: always_on
description: Synapse is a **decentralized P2P inference protocol for Mixture-of-Experts (MoE) models**. It coordinates consumer GPUs into a distributed inference swarm — miners contribute idle hardware, clients consume frontier AI through an OpenAI-compatible API. No datacenter, no gatekeeper, no rate limits.
---

# Repository Guidelines

## Project Overview

Synapse is a **decentralized P2P inference protocol for Mixture-of-Experts (MoE) models**. It coordinates consumer GPUs into a distributed inference swarm — miners contribute idle hardware, clients consume frontier AI through an OpenAI-compatible API. No datacenter, no gatekeeper, no rate limits.

**Multi-language monorepo:** Rust core (P2P + gateway), Python vLLM runtime (subprocess), Solidity staking contracts (L2).

## Architecture & Data Flow

```
Client → axum Gateway (Rust, :8000) → Swarm Core (Rust)
                                          │
                                    libp2p Kademlia DHT
                                          │
                                    Compute Nodes (Python vLLM)
                                          │
                                    L2 Smart Contracts (Solidity)
```

- **Gateway** (`synapse-core/src/gateway/`): axum HTTP server with OpenAI-compatible endpoints. Market maker pricing. Routes requests to swarm.
- **Swarm Core** (`synapse-core/src/swarm/`): Consensus (ensemble voting + statistical audit), Speculative engine (realtime), DAG engine (batch).
- **DHT** (`synapse-core/src/dht/`): Kademlia-based expert registry, co-activation heat map, node discovery.
- **Economic** (`synapse-core/src/economic/`): Reputation scoring (0-1000, 4 tiers), graduated slashing, route assembly.
- **Compute Node** (`synapse-runtime/`): Python subprocess communicating via Unix socket + protobuf through `InferencePort` trait. V1: vLLM backend. V2+: llama.cpp, SGLang.
- **Contracts** (`contracts/stake/`): StakeManager.sol — USDC staking, flagging, graduated slashing, banning.

## Non-Negotiable Design Principles

These apply to every line of code. No exceptions.

- **DDD: Pure domain layer** — zero I/O, zero framework deps, zero crypto. Domain types are plain structs/enums. I/O boundaries are traits (ports) in domain modules. Infrastructure adapters live in `infrastructure/` subdirectories.
- **Clean Architecture: Dependencies point inward** — Presentation (axum) → Ports (traits) → Infrastructure (adapters) → Domain. Domain never imports infrastructure.
- **TDD: Red-Green-Refactor** — Write the failing test first, confirm it fails, then implement, then refactor. Tests inline with source at `#[cfg(test)] mod tests`.
- **Clean Code: Every public item gets `///` doc comments.** Test names describe the scenario. No dead code. `thiserror` for errors, never manual `Display`/`Error`. Conventional Commits.

### Two Swarm Modes
- **Speculative Swarm (realtime):** N nodes run full model independently, majority vote per token. Latency = single-node latency.
- **Swarm DAG (batch):** True expert distribution. Nodes hold 2-5 experts each. Requests flow through expert graph.

## Key Directories

```
synapse/
├── synapse-core/            # Rust — single crate, single binary
│   ├── src/
│   │   ├── main.rs          #   Binary entrypoint (axum + swarm + DHT)
│   │   ├── gateway/         #   axum HTTP: api, catalog, pricing, router, middleware
│   │   ├── identity/        #   NodeId, KeyPair, Node aggregate
│   │   ├── model/           #   ModelId, ExpertId, Catalog
│   │   ├── swarm/           #   Consensus, Speculative engine, DAG engine
│   │   ├── economic/        #   Reputation, Pricing, Stake management
│   │   ├── transport/       #   WebRTC, Signalling
│   │   └── dht/             #   Kademlia, Expert registry, Bootstrap
│   └── proto/               #   Protobuf schemas (8 message types)
├── synapse-runtime/         # Python — vLLM adapter (subprocess)
│   └── synapse_runtime/     #   Package source
├── contracts/stake/         # Solidity — StakeManager + Hardhat
│   ├── src/                 #   StakeManager.sol
│   └── test/                #   Hardhat tests
├── config/
│   ├── models.toml          #   Curated catalog (Kimi K3, Mixtral, etc.)
│   └── default.toml         #   Node defaults (VRAM, pricing, STUN)
├── features/                #   Gherkin BDD specs
├── .github/workflows/       #   CI (7 jobs)
└── docs/superpowers/        #   Design spec + implementation plan
```

## Development Commands

```bash
# Rust
cargo build --release              # Build single binary
cargo test                         # Run all Rust tests
cargo fmt --check                  # Check formatting
cargo clippy -- -D warnings        # Lint
cargo llvm-cov --fail-under-lines 80  # Coverage check
cargo mutants -- --workspace       # Mutation testing
cargo deny check                   # License + dependency audit
cargo audit                        # CVE check

# Python
cd synapse-runtime && ruff check . && ruff format --check .
cd synapse-runtime && python -m pytest tests/ -v
cd synapse-runtime && pip-audit

# Solidity
cd contracts/stake && npx hardhat compile && npx hardhat test
cd contracts/stake && npx solhint 'src/**/*.sol'

# Everything at once (PR gate)
make gauntlet
```

## Code Conventions & Common Patterns

### Rust

- **Edition:** 2024, pinned to Rust 1.97 via `rust-toolchain.toml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonygiomarxdev/synapse](https://github.com/antonygiomarxdev/synapse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
