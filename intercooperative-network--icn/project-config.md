---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working with the ICN (Intercooperative Network) codebase.
---

# GitHub Copilot Instructions for ICN

This file provides guidance to GitHub Copilot when working with the ICN (Intercooperative Network) codebase.

## Absolute Rules (must follow)

1. **Read `AGENTS.md` first** for operating mode, invariants, and change routing.
2. **Never weaken safety to fix tests**: Do not relax validation, trust gates, signature checks, encoding rules, or determinism requirements.
3. **Verify before claiming**: Do not claim tests passed without showing output. Run the actual commands.
4. **Follow change routing**: Run the right checks for what you touched (see `AGENTS.md`).
5. **Docs must match code**: If you change semantics, update the relevant doc/spec in the same PR.

> Custom agents are available in `.github/agents/` - use `@icn-orchestrator` to route multi-subsystem requests.

> For path-specific instructions (Rust, web, SDK, docs), see `.github/instructions/` directory.

## Project Overview

ICN is a substrate daemon for the cooperative internet. It is **not** a blockchain or federation server - it's a P2P coordination layer providing:

- **Identity Layer**: Decentralized identifiers (DIDs) with Ed25519 cryptography
- **Trust Graph**: Web-of-participation based trust computation
- **Networking**: QUIC/TLS secure sessions with mDNS discovery
- **Cooperative Contracts**: CCL (Cooperative Contract Language) execution
- **Mutual Credit Ledger**: Double-entry accounting with Merkle-DAG
- **P2P Coordination**: Gossip protocol with trust-gated topics
- **Distributed Compute**: Trust-gated CCL execution with intelligent scheduling
- **Governance**: Democratic proposals and voting primitives
- **Gateway API**: REST + WebSocket API for cooperative applications
- **Cooperative Management**: Lifecycle, membership, and multi-stakeholder governance
- **Federation**: Inter-cooperative coordination and cross-coop settlements
- **Privacy**: Encrypted metadata, onion routing, traffic obfuscation
- **Post-Quantum Security**: Hybrid cryptography for long-term protection
- **SDIS Integration**: Sovereign Digital Identity with VUI and zero-knowledge proofs
- **Byzantine Tolerance**: Misbehavior detection and reputation-based security

## Repository Structure

```
icn/
├── icn/                    # Main Rust workspace
│   ├── crates/            # Core library crates
│   │   ├── icn-core/      # Actor runtime & supervisor
│   │   ├── icn-identity/  # DID generation & keystore
│   │   ├── icn-trust/     # Trust graph computation
│   │   ├── icn-net/       # QUIC/TLS networking
│   │   ├── icn-gossip/    # Topic-based gossip protocol
│   │   ├── icn-ledger/    # Mutual credit accounting
│   │   ├── icn-ccl/       # Contract language interpreter
│   │   ├── icn-store/     # Persistent storage (Sled)
│   │   ├── icn-rpc/       # gRPC API server
│   │   ├── icn-gateway/   # REST + WebSocket API
│   │   ├── icn-governance/# Governance primitives
│   │   ├── icn-compute/   # Distributed compute layer
│   │   ├── icn-obs/       # Metrics & observability
│   │   ├── icn-coop/      # Cooperative management & lifecycle
│   │   ├── icn-community/ # Community structures & civic engine
│   │   ├── icn-entity/    # Unified entity model (individuals/coops/federations)
│   │   ├── icn-federation/# Inter-cooperative coordination
│   │   ├── icn-privacy/   # Privacy primitives & metadata protection
│   │   ├── icn-security/  # Byzantine fault detection & reputation
│   │   ├── icn-crypto-pq/ # Post-quantum hybrid cryptography
│   │   ├── icn-steward/   # SDIS steward network & VUI computation
│   │   ├── icn-zkp/       # Zero-knowledge proofs for SDIS
│   │   ├── icn-time/      # Clock synchronization (Rough Time Protocol)
│   │   ├── icn-snapshot/  # State snapshots for restarts
│   │   ├── icn-api/       # API types and definitions
│   │   ├── icn-encoding/  # Serialization utilities
│   │   └── icn-testkit/   # Testing utilities
│   └── bins/              # Binaries
│       ├── icnd/          # ICN daemon
│       ├── icnctl/        # CLI management tool
│       └── icn-console/   # Interactive TUI for cooperative management
├── docs/                  # Comprehensive documentation
├── deploy/                # Kubernetes & deployment configs
├── web/                   # Web UIs (pilot-ui, etc.)
├── sdk/                   # Client SDKs (TypeScript, etc.)
└── examples/              # Usage examples
```

## Current Working Context

- See docs/STATE.md and docs/TODO.md for current status, known issues, and priorities.
- Rust toolchain note: wasmtime/cranelift currently require rustc 1.89.0; upgrade toolchain or pin compatible versions before running cargo.

## Development Workflow

### Build & Test Commands

All commands must be run from the `icn/` directory:

```bash
# Build everything
cargo build

# Build release binaries
cargo build --release

# Run all tests
cargo test

# Run tests for a specific package
cargo test -p icn-gossip

# Run a specific test by name
cargo test test_two_node_convergence

# Linting
cargo clippy -- -D warnings

# Formatting
cargo fmt

# Run the daemon
./target/debug/icnd

# Use the CLI
./target/debug/icnctl status
```

### Code Quality Standards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InterCooperative-Network/icn](https://github.com/InterCooperative-Network/icn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
