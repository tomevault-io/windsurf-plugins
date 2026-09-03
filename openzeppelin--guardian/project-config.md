---
trigger: always_on
description: This file is the operational guide for coding agents working in this repository.
---

# AGENTS.md - Guardian

This file is the operational guide for coding agents working in this repository.
It is optimized for safe, cross-layer changes in a multi-language codebase.

## 1) System Shape (Read First)

Work from the bottom up:

1. `crates/server` (core system of record — gRPC, HTTP REST, operator dashboard, optional EVM)
2. Base clients (one per server surface)
   - `crates/client` + `packages/guardian-client` (Rust gRPC + TS HTTP REST for the per-account API)
   - `packages/guardian-operator-client` (TS HTTP client for the operator dashboard surface)
   - `packages/guardian-evm-client` (TS HTTP client for the feature-gated EVM surface)
3. Higher-level SDKs
   - `crates/miden-multisig-client` + `packages/miden-multisig-client` (multisig flow on top of base clients)
4. `examples/` (verification and debugging surfaces)
   - `examples/demo` = CLI/TUI multisig flow (Rust)
   - `examples/web` = browser multisig flow (TS)
   - `examples/rust` = low-level Rust integration examples
   - `examples/smoke-web` = TS multisig smoke harness
   - `examples/operator-smoke-web` = operator dashboard auth + account-API smoke harness
   - `examples/evm-smoke-web` = EVM proposal lifecycle smoke harness

If a behavior changes in a lower layer, verify and propagate impact upward across every relevant base client.

## 2) Repo Map

- `crates/server`: GUARDIAN server — gRPC + HTTP + `/dashboard/*` + feature-gated `/evm/*`, storage, metadata, auth, canonicalization jobs
- `crates/client`: Rust gRPC client SDK (per-account API)
- `packages/guardian-client`: TS HTTP REST client SDK (per-account API)
- `packages/guardian-operator-client`: TS HTTP client for the operator dashboard surface (`/dashboard/*`); session-cookie auth, permission vocabulary
- `packages/guardian-evm-client`: TS HTTP client for the feature-gated EVM surface (`/evm/*`)
- `crates/miden-multisig-client`: Rust multisig SDK on top of Miden + GUARDIAN
- `packages/miden-multisig-client`: TS multisig SDK on top of Miden + GUARDIAN
- `crates/shared`: shared Rust primitives/utilities
- `spec/`: system and protocol-level behavior docs
- `docs/`: contributor- and operator-facing documentation hub (start at `docs/CONCEPTS.md`)
- `infra/`: Terraform for the AWS reference deployment
- `examples/`: validation apps and reference flows

## 3) Core Change Rules

1. Preserve protocol compatibility unless explicitly asked to break it.
2. Treat server contract changes as multi-package changes by default.
3. Update tests in the layer where behavior changes, plus at least one upstream consumer.
4. Prefer minimal, surgical edits over broad refactors.
5. Do not introduce silent behavior drift between Rust and TypeScript clients.
6. Do not add backward-compatibility layers, migrations, or fallback behavior unless the task explicitly requires them.

## 4) Contract-Change Workflow (Mandatory)

Use this when changing endpoints, payloads, status enums, signatures, or auth behavior.

1. Update server contract source first:
   - gRPC: `crates/server/proto/guardian.proto`
   - HTTP shapes/serialization in server services and API modules
   - HTTP OpenAPI: any change to an HTTP handler, its request/response/query/path
     types, auth behavior, or a feature-gated route MUST update the `#[utoipa::path]`
     annotation and `#[derive(utoipa::ToSchema)]`/`IntoParams` derives, then
     regenerate the committed specs:
     `cargo run --features evm --bin gen-openapi -- docs`
     (CI fails on drift via `cargo run --features evm --bin gen-openapi -- --check docs`).
2. Update Rust client compatibility:
   - `crates/client` (proto, request/response mapping, auth/signature handling)
3. Update TS client compatibility (each affected surface):
   - `packages/guardian-client/src/server-types.ts` — per-account API
   - `packages/guardian-operator-client` — when `/dashboard/*` shapes, permission vocabulary, or session/cookie semantics change
   - `packages/guardian-evm-client` — when `/evm/*` shapes change
   - request/response adapters and tests in each touched package
4. Update multisig SDK layers if proposal/state shape changed:
   - `crates/miden-multisig-client`
   - `packages/miden-multisig-client`
5. Validate via examples (use the matching smoke harness for the surface you changed):
   - `examples/demo` for the Rust CLI multisig flow
   - `examples/web` / `examples/smoke-web` for the TS multisig flow
   - `examples/operator-smoke-web` for dashboard / operator-client changes
   - `examples/evm-smoke-web` for EVM proposal flow changes
6. Run targeted tests, then broader suite.

## 5) Layer-Specific Guidance

### Server (`crates/server`)

- Keep business logic in `src/services/`; keep transport logic thin in `src/api/`.
- Respect auth expectations: unauthenticated-only endpoints must remain explicit.
- Document every HTTP handler with `#[utoipa::path]` (method, path, params, body,
  per-status responses, and `security(...)` for authenticated routes) and derive
  `utoipa::ToSchema`/`IntoParams` on its wire types. Regenerate `docs/openapi*.json`
  after any HTTP contract change; do not hand-edit endpoint shapes into `spec/api.md`.
- Maintain storage/metadata backend parity (filesystem/postgres where applicable).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenZeppelin/guardian](https://github.com/OpenZeppelin/guardian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
