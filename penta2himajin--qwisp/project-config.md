---
trigger: always_on
description: Qwisp is a single-model-specialised local inference engine for **Qwen3.6-35B-A3B (MoE)** on
---

# Qwisp

## Overview

Qwisp is a single-model-specialised local inference engine for **Qwen3.6-35B-A3B (MoE)** on
Apple Silicon (MLX). It streams MoE expert weights from flash and keeps only the active slice
resident, extending the reachable model size on RAM-constrained Macs. The decode core is
**Seedless** — a raw-Metal engine (persistent buffers, hand-issued command buffers, int32
readback) that runs *outside* the MLX op-graph, peer to "MLX" as a backend.

Positioning: fastest practical-accuracy local LLM for power-users + researchers, with
**bit-exact lossless** (strict L1: reproduces the quantised greedy token stream) exposed as an
option. See @README.md.

> **Status: productization.** The research phase is closed and the engine is frozen. Work is
> converging the ~29K-line research PoC into a shippable OpenAI-compatible local server. Volatile
> campaign state lives in `HANDOFF.md` and `notes/`, not here.

## Project Structure

```
swift/            # the product — Swift package
  Sources/QwispCore/    # Tell runtime + Seedless engine (raw-Metal forward, arena/streaming, spec-verify) + locked tests
  Sources/qwisp/        # OpenAI server + `qwisp chat` CLI + tokenizer (swift-transformers)
  Sources/qwisp-poc/    # bench/gate binary (RAWTESTS + bench harness)
scripts/          # shell gate + benchmark scripts
oracle/           # Python reference/bench oracle (bit-compare only; NEVER in the serving path)
notes/            # engine design rationale (referenced by number from source comments)
docs/             # process docs (handoff-protocol, i18n-policy)
refs/             # canonical measurement refs (raw-greedy) — GITIGNORED, regenerate locally
```

The boundary that matters: **Swift = product + engine; Python = reference oracle only.** The
server is Swift, in-process — the engine holds GBs of resident Metal buffers a language boundary
cannot cheaply reach.

## Development Setup

- **Xcode + Metal Toolchain** required (raw-Metal kernels). SourceKit shows "No such module 'MLX'"
  on QwispCore files — LSP-only noise; `xcodebuild` is the truth.
- Model: a Qwen3.6-35B-A3B MTPLX checkpoint; point `QWISP_MODEL` at its directory.
- Python reference oracle needs an MLX-capable python (numpy/safetensors/mlx_lm), not Homebrew
  python3 — see `oracle/README.md`.

## Build & Test

```bash
# build (Release; Metal Toolchain required; ~minutes)
#   scheme `qwisp` = product (server + CLI) ; scheme `qwisp-poc` = bench/gate binary
cd swift && xcodebuild build -scheme qwisp -configuration Release \
  -destination 'platform=macOS' -derivedDataPath ./.xcode-build-rel \
  -skipPackagePluginValidation

# correctness gates (must stay green through every commit)
scripts/test_raw.sh          # → RAWTESTS 89/89     (engine, GPU, no model)
scripts/test_bench_batch.sh  # → BENCHBATCHTEST PASS (fixture, no GPU)
scripts/test_tokenizer.sh    # → TOKTEST 3/3        (needs model tokenizer files)
scripts/test_completion.sh   # → COMPTEST 4/4       (needs model tokenizer files)
```

`refs/*.safetensors` are gitignored. A fresh checkout / `git clean` loses them and makes the
strict fidelity gate false-red on longctx/shortnl — regenerate per `HANDOFF.md` before trusting a
red strict cell.

## Development Principles

- **Lossless is defined at L1: bit-exact reproduction of the quantised greedy token stream.** The
  strict path is the reference; `bolt`/near-lossless is an opt-in speed tier. Never weaken the
  lossless definition to make a number look better.
- **RAWTESTS 89/89 is the campaign-wide safety gate.** It must stay green through every
  delete/rename/refactor commit. A red gate blocks the commit, not the other way around.
- **Predictive levers lose to mechanical levers on the same slack** (engine doctrine, measured
  repeatedly). Don't re-propose prediction/prefetch schemes as speedups without new measurement.
- **Measure before implementing.** Heavy Metal/kernel changes get a Python physical-plausibility
  check first (the research phase burned weeks on plausible-but-wrong kernel ideas).

## Architectural Boundaries

- The Seedless decode core does **not** use the MLX op-graph — its speed comes from raw command
  buffers and persistent buffers. MLX compatibility is coarse (load + generate + tier via a
  backend protocol), never op-level.
- Python stays a reference/bit-compare oracle. It is never on the serving path.
- `refs/` is the canonical raw-greedy measurement set — regenerate from Swift raw greedy only,
  never from MLX or a bootstrap.

## Prohibitions

1. Do not regenerate `refs/*.safetensors` from MLX or any non-raw-greedy source.
2. Do not weaken, skip, or delete the `WRITE-LOCKED` tests in
   `swift/Sources/QwispCore/SeedlessVerifyTests.swift` (guarded by the `total = N` counter; extending the
   suite with new locked tests bumps N — weakening existing ones never does). They are the lossless safety net.
3. Do not rewrite the shipped forward path (SeedlessEngine / SeedlessMetalForward / SeedlessFusedVerify /
   Tell / ExpertArena / ExpertSource + model layers). It is frozen — refactor/rename only, never rewrite.
4. Do not rewrite `main`'s history. Work on a topic branch (`claude/<topic>`, etc.) and open a PR to `main`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [penta2himajin/qwisp](https://github.com/penta2himajin/qwisp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
