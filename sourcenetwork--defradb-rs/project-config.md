---
trigger: always_on
description: **The Go compatibility baseline lives in `crates/defra-version/src/lib.rs`, not
---

# Development Principles

## 0. The North Star: 1.0 Release

**The Go compatibility baseline lives in `crates/defra-version/src/lib.rs`, not
here.** Three constants define it: `GO_COMPAT_BRANCH`, `GO_COMPAT_COMMIT`, and
`GO_COMPAT_TAG`. An empty tag means CI builds the pinned commit from source
instead of downloading a release binary. `.github/workflows/ci.yml` greps those
constants directly, so they are the single source of truth — read them rather
than quoting a release number, which goes stale on every baseline bump.

Go parity is achieved across CLI, HTTP API, GraphQL query engine, and P2P replication. We now validate with Rust-native integration tests that exercise the full stack, and are building Rust-specific features (Iroh transport, BM25 full-text search, Postgres wire protocol, WASM client).

### What "Parity" Means

- Same GraphQL query → Same results (field values, ordering, errors)
- Same document content → Same document ID (content-addressed CIDs)
- Same CRDT operations → Same merged state (convergence)
- Same P2P protocol → Nodes can discover, connect, and replicate
- Same CLI commands → Same behavior and output
- Same HTTP API → Same wire format and response structure

---

## 1. Information Hygiene

This codebase is designed for **AI-human pair programming**. Every structural choice optimizes for **rapid context acquisition**.

**Context clarity is oxygen for productive collaboration.**

## 2. Temporal Boundaries

| Zone | Contains | Lives in |
|------|----------|----------|
| **Past** | How we got here | Git history, closed issues/PRs |
| **Present** | What the code does now | Working tree |
| **Future** | What we might do next | GitHub issues |

**No commented-out code. No TODO comments (create issues instead). No speculative docs.**

## 3. No Documentation Files

Only allowed: `README.md`, `CLAUDE.md`, `Cargo.toml` files.

No `ROADMAP.md`, `DEVELOPMENT.md`, `docs/` directories, or planning documents.

## 4. File Organization

**One concept per file. Small files over large files.**

### Crate Structure

```
crates/
├── acp/                # Access Control Policy
├── blockstore/         # IPLD block storage
├── cli/                # Command-line interface
├── crdt/               # CRDT implementations
├── crypto/             # Cryptographic operations
├── cursor/             # Opaque cursor token codec for GraphQL pagination
├── datastore/          # Data persistence abstractions
├── db/                 # Database core, one module per execution role:
│                       #   txn/ (the transaction seam), read/, write/,
│                       #   collection/, block/, definition/, access/,
│                       #   database/, docid/, downsample/, view/,
│                       #   event/, error/, plus the folded former crates:
│                       #   backup/, block/builder/, index/, merge/, nac/,
│                       #   search/. src/ holds only lib.rs; every test
│                       #   lives in db/tests/.
├── defra-core/         # Core types and traits
├── defra-node/         # Reusable embedded node builder
├── defra-version/      # Version metadata and Go compat tracking
├── document/           # Document handling
├── embedded/           # Embedded/mobile node assembly
├── events/             # Pub/sub event bus (subscriptions)
├── ffi/                # C-compatible FFI bindings
├── http/               # HTTP API server
├── identity/           # Identity and JWT management
├── keyring/            # Key storage
├── kms/                # DEK generation, wrapping, and distribution under NAC/DAC
├── lens/               # Schema migration via WASM transforms
├── orbis/              # Threshold BLS signing (Orbis ring client)
├── p2p/                # P2P networking (libp2p + optional Iroh)
├── p2p-adapter/        # P2P adapters for HTTP-facing operations
├── pg-compat/          # Postgres wire protocol compatibility
├── query/              # Query engine (GraphQL, BM25)
├── replication-filter/ # Query-filter-backed replication matcher
├── schema/             # Schema validation
├── sourcehub/          # On-chain ACP client (Cosmos/EVM)
├── storage/            # Storage backends (lark, redb, fjall, rocksdb, memory)
├── telemetry/          # OpenTelemetry exporter setup
├── wasm/               # Browser client (WebAssembly)
└── zanzibar/           # Google Zanzibar permission engine

tools/
├── apple/                  # Apple embedding: .xcframework build + Swift import smoke test
├── ffi-test/               # FFI compatibility testing against Go
├── hf_embedding_server.py  # Local HuggingFace embedding server for embedding benchmarks
├── integration-test/       # Rust-native integration tests (primary validation)
├── lens-host/              # Standalone WASM Lens transform runner (JSON stdin → stdout)
├── otel-smoke/             # OTLP exporter smoke tests against a Compose-run collector
└── pg-compat-harness/      # Drizzle ORM harness for the Postgres wire protocol
```

Only `ffi-test`, `integration-test`, and `lens-host` are workspace members; the
rest are scripts and non-Rust harnesses.

### File Size Guidelines

- Under 200 lines: Fine
- 200-400 lines: Check if doing one thing
- Over 400 lines: Consider splitting

## 5. Naming Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sourcenetwork/defradb.rs](https://github.com/sourcenetwork/defradb.rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
