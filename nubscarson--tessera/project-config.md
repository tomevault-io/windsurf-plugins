---
trigger: always_on
description: Tessera is an **anonymous-credentials trust layer**: admit a web request on an
---

# AGENTS.md — Tessera contributor & agent guide

Tessera is an **anonymous-credentials trust layer**: admit a web request on an
unlinkable, rate-limited credential — **never on its source IP**. The
cryptographic core is a from-scratch, vector-proven implementation of IETF
**Anonymous Rate-Limited Credentials (ARC) over NIST P-256** — a
keyed-verification anonymous credential (KVAC) scheme — wrapped in a small
network: an issuance authority, a client, a 2-hop split-trust access loop
(blind relay + credential-gated CONNECT exit), and an optional EVM settlement
tier.

This file is the canonical, tool-agnostic guide for both engineers and AI
agents. Read it before changing anything.

> ### Posture (load-bearing — do not soften)
> **Research-grade. UNAUDITED.** Not constant-time end-to-end. Classical
> discrete-log over P-256 only — **no post-quantum claim** (Shor breaks it).
> Smart contracts are **TESTNET-ONLY**. Do **not** describe any part of Tessera
> as audited, secure, production-ready, or as delivering private/uncensorable
> clearnet access. The honest headline is: *a vector-proven ARC/P-256
> keyed-verification implementation plus supporting network* — not "anonymity."
> The project's credibility is its calibrated candor; every "UNAUDITED" /
> "testnet-only" / "cost knob, not Sybil resistance" / "no post-quantum claim"
> qualifier is intentional. Keep them.

---

## 1. Repo layout

```
crates/        Rust workspace members + two excluded sub-workspaces
contracts/     Foundry / Solidity settlement layer (TESTNET-ONLY)
circuits/      circom R_dec circuit + snarkjs build tooling (TEST-ONLY setup)
docs/          The primary deliverable surface alongside the code (read these)
deploy/        docker-compose + dstack deployment scaffolding
fuzz/          nightly cargo-fuzz targets (excluded sub-workspace)
target/        build output (gitignored)
```

Root long-form docs: `README.md`, `AUDIT.md`, `GOAL.md`, `SECURITY.md`,
`CONTRIBUTING.md`, `CHANGELOG.md`, `DEMO.md`. `Cargo.lock` is committed; license
is `MIT OR Apache-2.0`.

### Workspace crates (members of the root `[workspace]`)

| Crate | Role |
| --- | --- |
| `tessera-arc` | The crypto core: from-scratch IETF ARC over P-256 (KVAC). Issuance + presentation + the deterministic double-spend tag. Keyed-verification — **no public verifiability**. |
| `tessera-issuer` | Credential **authority** node: gates who can *obtain* a credential via a PoW gate (a cost knob) or a paid on-chain TokenMint gate; convergent shared-key bootstrap. |
| `tessera-client` | Thin, transport-agnostic client: drives issuance and mints one fresh unlinkable presentation per request as the hex `Tessera-Presentation` header. Owns no crypto. |
| `tessera-origin` | Server-side admission **guard** (`OriginGuard::check`) + spent-tag stores + optional `tower` middleware (off by default). Source IP is never an input. |
| `tessera-proxy` | Credential-gated CONNECT **exit**: admits on a credential, tunnels opaque bytes (direct or via Tor SOCKS5), applies per-egress human-volume shaping. |
| `tessera-relay` | The credential-**blind** first hop + the 2-hop split-trust loop; ships the relay node binary and the local `tessera-client` proxy binary. |
| `tessera-channel` | Off-chain Spilman payment-channel **protocol** state machine (Phase 2a, plain crypto). **Not ZK, moves no money.** secp256k1 (EVM-native), the workspace exception. |
| `tessera-demo` | Runnable end-to-end narration binary (localhost; `--tor`; `--serve`). Demo tooling, not a library. |

### Excluded sub-workspaces (own manifest, own `Cargo.lock`, own CI job)

Listed in root `Cargo.toml` `[workspace].exclude` so their special deps can
never perturb the host MSRV-1.74 / clippy / test / doc gates. They are **not**
workspace members — you must pass `--manifest-path`.

| Path | Role |
| --- | --- |
| `crates/tessera-wasm` | `tessera-arc` + `tessera-client` compiled to `wasm32` via wasm-bindgen (browser client) + an MV3 extension **scaffold**. Excluded for the wasm32-only `getrandom` `js` dep. |
| `crates/tessera-tower-demo` | Runnable axum origin+issuer using `tessera-origin`'s `TesseraLayer`. Excluded for the axum+tokio deps. |
| `fuzz/` | Nightly-only cargo-fuzz targets. |

### Non-Rust components

- `contracts/` — three Solidity files: `TokenMint.sol` (ETH-paid mint rail, the
  leaner default), `ChannelRegistry.sol` (the optional-advanced on-chain
  "court"), `RDecVerifier.sol` (**generated**, TEST-ONLY trusted setup).
  Dependency-free Foundry project (vendored `test/Std.sol`, pinned solc 0.8.24).
- `circuits/` — the R_dec decrement circuit and its build tooling. Build scratch
  (`build/`, `node_modules`, `*.ptau`, `*.zkey`, `*.wtns`) is gitignored and
  regenerable; **TEST-ONLY**.

---

## 2. Build, test & lint

Toolchain is **stable**, pinned via `rust-toolchain.toml` (rustfmt + clippy
components). **MSRV is 1.74** (`rust-version = "1.74"`). `Cargo.lock` is
committed for reproducible, auditable tooling — keep it. Edition is 2021.

Run the suite rather than trusting any hard-coded test/fuzz **counts** in the
docs — several docs carry stale numbers; `AUDIT.md` + `docs/CEILING_PROGRESS.md`
are the closest to current, but re-measure from source if it matters.

**Main workspace** (cwd `/home/nubs/tessera`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NubsCarson/tessera](https://github.com/NubsCarson/tessera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
