---
trigger: always_on
description: Colibri Stateless is a high-performance prover/verifier for Ethereum and Layer-2 chains (OP-Stack). The core library is written in C with bindings for JavaScript/TypeScript, Python, Kotlin/Java, and Swift. It enables ultra-light clients that cryptographically verify blockchain data without storing full state -- the verifier only stores sync committee state (~27h rotation).
---

# Colibri Stateless - Agent Guide

Colibri Stateless is a high-performance prover/verifier for Ethereum and Layer-2 chains (OP-Stack). The core library is written in C with bindings for JavaScript/TypeScript, Python, Kotlin/Java, and Swift. It enables ultra-light clients that cryptographically verify blockchain data without storing full state -- the verifier only stores sync committee state (~27h rotation).

**License**: MIT (core), PolyForm Noncommercial (server component `src/server/`).

## Architecture Overview

```
                  ┌─────────────────────────────────────────────┐
                  │            Bindings / Host System            │
                  │  (JS/TS, Python, Kotlin, Swift, CLI, HTTP)   │
                  └───────────────┬─────────────┬───────────────┘
                                  │             │
                        ┌─────────▼──┐    ┌─────▼────────┐
                        │   Prover   │    │   Verifier   │
                        │ prover.h   │    │  verify.h    │
                        └─────┬──────┘    └──────┬───────┘
                              │                  │
                  ┌───────────▼──────────────────▼───────────┐
                  │          State Machine (state.h)          │
                  │   C4_PENDING / C4_SUCCESS / C4_ERROR      │
                  └───────────────────┬──────────────────────┘
                                      │
                  ┌───────────────────▼──────────────────────┐
                  │         Chain Modules (Plugin System)     │
                  │   chains/eth/  │  chains/op/              │
                  └───────────────────┬──────────────────────┘
                                      │
                  ┌───────────────────▼──────────────────────┐
                  │              Utilities                    │
                  │  ssz.h  bytes.h  crypto.h  json.h        │
                  └──────────────────────────────────────────┘
```

### Core Concepts

- **Prover**: Collects blockchain data from RPC/Beacon API nodes and creates cryptographic proofs for the validity of RPC responses.
- **Verifier**: Validates proofs using only the current sync committee state (no full node needed). Almost stateless.
- **State Machine**: `c4_state_t` manages pending `data_request_t` entries and error messages. It is embedded in `prover_ctx_t` and `verify_ctx_t` (as `ctx->state`). Functions return `c4_status_t`: `C4_PENDING` when external data is needed, `C4_ERROR` on failure, `C4_SUCCESS` when done. The host system is responsible for fetching data for pending requests and setting responses. `TRY_ASYNC()` is used broadly for error propagation, not just in async contexts.
- **Chain Modules**: Ethereum and OP-Stack are implemented as plugins registered via CMake (`add_verifier()` / `add_prover()`). At build time, CMake generates dispatcher headers (`verifiers.h`, `provers.h`).
- **SSZ**: All proofs and data types use Simple Serialize (SSZ) encoding. Types are defined declaratively in C using `ssz_def_t` arrays.

### Request Lifecycle

1. Host creates a prover or verifier context with RPC method, params, and chain ID.
2. Execute function returns `C4_PENDING` with data requests.
3. Host fetches external data (RPC, Beacon API) and sets responses via `c4_req_set_response()`.
4. Repeat until `C4_SUCCESS` (proof/result ready) or `C4_ERROR`.

## Directory Structure

| Directory | Purpose | Details |
|-----------|---------|---------|
| `src/` | Core C library | See [src/AGENTS.md](src/AGENTS.md) |
| `src/verifier/` | Verification engine | `verify.h` is the main API |
| `src/prover/` | Proof generation engine | `prover.h` is the main API |
| `src/chains/eth/` | Ethereum chain module | See [src/chains/eth/AGENTS.md](src/chains/eth/AGENTS.md) |
| `src/chains/op/` | OP-Stack chain module | See [src/chains/op/AGENTS.md](src/chains/op/AGENTS.md) |
| `src/util/` | Utilities (SSZ, bytes, crypto, state, JSON) | See [src/util/AGENTS.md](src/util/AGENTS.md) |
| `src/server/` | HTTP prover server (libuv/llhttp). Do not block the event loop -- use `REQUEST_WORKER_THREAD` for CPU work | See [src/server/AGENTS.md](src/server/AGENTS.md) |
| `src/cli/` | CLI tools (prover, verifier, ssz) | Three executables |
| `bindings/` | Language bindings | See [bindings/AGENTS.md](bindings/AGENTS.md) |
| `bindings/colibri.h` | Public C API for all bindings | JSON-based status protocol |
| `libs/` | Bundled third-party libraries | blst, evmone, libuv, llhttp, zstd, mcl, etc. |
| `test/` | Tests (Unity framework) | See [test/AGENTS.md](test/AGENTS.md) |
| `scripts/` | Build and doc scripts | `doc/`, `create_test.sh`, coverage, valgrind |
| `installer/` | Platform installers | Linux, macOS, Windows, Homebrew, PPA |
| `.github/workflows/` | CI/CD pipelines | cmake.yml, bindings, release, CodeQL |

<!-- AUTO:DIRECTORY_MAP:START -->
- `bindings/` (6 .c, 11 .h) -- Language Bindings
  - `bindings/dart/` (1 .c, 8 .h) -- Colibri Dart Bindings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [corpus-core/colibri-stateless](https://github.com/corpus-core/colibri-stateless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
