---
trigger: always_on
description: This file provides context for AI agents working with the IOTA Rust SDK repository.
---

# AGENTS.md

This file provides context for AI agents working with the IOTA Rust SDK repository.

## Project Overview

The **IOTA Rust SDK** is a modular software development kit for integrating with the IOTA blockchain. IOTA is a next-generation smart contract platform powered by Move.

**Key Design Goals:**

- Modularity: Users only pay for features they use
- Lightweight: Minimal dependency footprint
- WASM Support: Libraries usable in browser environments
- Multi-language: FFI bindings for Go, Kotlin, Python, C#, Swift (via `uniffi`)

## Critical development notes

1. **NEVER make breaking changes** — this SDK is consumed externally. New fields must be optional, removals require a deprecation step first.
2. **NEVER disable or skip tests** — all tests must pass and stay enabled.
3. **NEVER use `#[allow(dead_code)]`, `#[allow(unused)]`, or other lint suppressions** to silence warnings — fix the underlying issue.
4. **Types in `iota-sdk-types` must stay BCS-compatible** — verify BCS and JSON round-trips when adding or changing a type. `u64` is serialized as a string in JSON for JS safety.
5. **Format and lint after every change** — `cargo +nightly fmt`, `dprint fmt`, and `make bindings-examples-format-check` for binding examples.
6. **Keep pull requests small** — prefer small, focused PRs over large ones. A small diff is easier to review, easier to revert, and less likely to introduce regressions.
7. **Split work into multiple PRs when possible** — if a change spans multiple concerns (e.g. a refactor plus a new feature, or changes across unrelated crates), split it into separate PRs. Land independent pieces incrementally rather than bundling them together. **Critically: when given multiple GitHub issues, ALWAYS create one PR per issue** — never bundle multiple issues into a single PR unless explicitly instructed or the issues are genuinely interdependent.
8. **Write only what the diff can't say** — PR descriptions, review comments and chat replies cover the reasoning and the high-level shape of a change, never a walkthrough of the diff. See [Writing style](#writing-style); this is the most frequently ignored rule in this file.
9. **Feature flags matter** — the umbrella `iota-sdk` gates everything behind features. Check what's enabled for the code you're modifying before assuming an item exists.
10. **NEVER hand-edit generated gRPC types** under `crates/iota-sdk-grpc-types/src/proto/` — they are build output. Changes go into the proto sources / `update_grpc_types.sh`.

## Repository Structure

```
crates/
├── iota-sdk/                       # Umbrella SDK that re-exports the other crates behind feature flags
├── iota-sdk-bcs-schema/            # Proc macro that generates BCS schema definitions (ABNF) from Rust types
├── iota-sdk-crypto/                # Signing traits (`IotaSigner`, `IotaVerifier`) and implementations (ed25519, secp256r1, secp256k1, bls12381, passkey)
├── iota-sdk-ffi/                   # FFI layer powering language bindings via `uniffi` (not published)
├── iota-sdk-graphql-client/        # Type-safe GraphQL RPC client using `cynic`
├── iota-sdk-graphql-client-build/  # Build-time GraphQL schema registration for `cynic` codegen
├── iota-sdk-grpc-client/           # gRPC client built on `tonic` (ledger, execution, state, move package services)
├── iota-sdk-grpc-proto-build/      # Build-time codegen for gRPC/protobuf types (`update_grpc_types.sh` regenerates from upstream protos)
├── iota-sdk-grpc-types/            # Generated gRPC/protobuf types
├── iota-sdk-transaction-builder/   # Fluent API for building transactions (online/offline modes)
└── iota-sdk-types/                 # Core blockchain types (Address, ObjectId, Transaction, Checkpoint, ...) — BCS-compatible

bindings/
├── csharp/                         # C# bindings
├── go/                             # Go bindings
├── kotlin/                         # Kotlin bindings
├── python/                         # Python bindings
├── swift/                          # Swift bindings
└── wasm/                           # WASM/TypeScript bindings (browser + Node)
```

The `iota-sdk` umbrella crate exposes the other crates via modules gated by feature flags: `crypto`, `graphql` (→ `graphql_client`), `grpc` (→ `grpc_client` + `grpc_types`), `move-types` (→ `move_types`), `txn-builder` (→ `transaction_builder`), and `types`. `grpc` and `move-types` are opt-in (not in `default`); `graphql`, `crypto`, `types`, `txn-builder` are on by default.

## Build & Test Commands

```bash
# Lint, format, tests
make test                        # Unit tests (nextest)
make test-docs                   # Doc tests
make test-with-localnet          # Tests requiring a running localnet
make clippy                      # Clippy
make fmt                         # Format Rust code (requires nightly)
make check-fmt                   # Verify Rust formatting
make bindings-examples-format        # Format the examples shipped with each binding
make bindings-examples-format-check  # Verify formatting of binding examples

# WASM
make wasm32          # Check that SDK crates compile to wasm32-unknown-unknown
make wasm            # Build the WASM/TypeScript bindings package

# FFI bindings
make bindings        # Build all bindings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iotaledger/iota-rust-sdk](https://github.com/iotaledger/iota-rust-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
