---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

The search runs on an **NVIDIA GPU** via a CUDA kernel linked into the Go binary
with cgo, so the build is driven by `make` (nvcc + cgo), not plain `go build`.

```bash
# Build (nvcc compiles the kernel -> static lib -> cgo links the Go binary)
make                       # produces ./bitcoin_finder (a Linux ELF binary)
make CUDA_ARCH=sm_75       # override GPU arch (default sm_86 = RTX 3050)

# Run (from project root; reads data/ relative to the cwd)
./bitcoin_finder

# Tests (must build the GPU lib first; `make test` does both)
make test
CGO_ENABLED=1 go test -count=1 -run TestName .   # single test, uncached

# Clean build artifacts
make clean
```

Building requires `nvcc` (CUDA toolkit), a native Linux Go toolchain, and gcc.
`CGO_ENABLED=1` is required. `go test`'s result cache does **not** track the cgo
static library, so after recompiling the kernel use `-count=1` to force re-runs.

The program must be run from the project root directory, as it opens data files
relative to the working directory (`data/hash160s.json`, `data/ranges.json`,
optionally `data/wallets.json`).

**Build environment note (WSL):** the GPU is reachable from WSL through the WSL
CUDA driver, so build and run the Linux binary inside WSL. The Windows `go.exe`
(under `/mnt/c/Program Files/Go`) fails on the Linux filesystem with `RLock
go.mod: Incorrect function`; use a native Linux Go (e.g. `mise install go@1.22`).
Cross-compiling the CUDA+cgo binary to a Windows `.exe` from WSL is impractical,
so the committed `bitcoin_finder.exe` (the old CPU-only Windows build) is stale.

## Purpose

This is a CLI tool for participating in the **Bitcoin Puzzle challenges** (the well-known 1000 BTC / 32 BTC puzzle transactions, where keys for puzzles 1–160 are deliberately constrained to known ranges and solving them is the intended goal). The 160 wallets and their key ranges in `data/` correspond to these puzzle entries.

**This code exists solely for the puzzle challenge. It must never be adapted or used to attack regular wallets that don't belong to the puzzle.**

## Architecture

The tool searches private keys within a puzzle's defined range to find the key matching the target puzzle wallet's Hash160 (RIPEMD-160 of SHA-256 of the compressed public key).

### Flow

1. `main.go` — entry point: loads data, prompts for wallet number (1–160), resolves the target Hash160 and key range, then calls `searchForPrivateKey`.
2. `search.go` — GPU search driver. Generates the random per-walk seeds, drives the CUDA kernel `roundsPerLaunch` rounds at a time, logs throughput every 10 seconds (both the interval rate and the cumulative average), and on match writes the private key to `found_key_<hash160prefix>.txt`. See **Search algorithm** below.
3. `gpu.go` — cgo bindings to the CUDA library (`gpu/secp256k1_gpu.{h,cu}`): device detection, the search lifecycle (`gpuSearchInit`/`run`/`free`), and `gpuHash160Batch` (used only by the correctness test).
4. `gpu/secp256k1_gpu.cu` — the CUDA kernel: 256-bit field arithmetic mod p (64-bit limbs + `__int128`), Jacobian EC point add/double + base-point scalar mult, SHA-256 and RIPEMD-160, plus the seed/step/hash kernels. `gpu/secp256k1_gpu.h` is the `extern "C"` interface.
5. `bitcoin.go` — CPU **reference** primitives used to validate the GPU: `privateKeyToHash160` (`btcsuite/btcd` secp256k1 + `btcutil.Hash160`) and `privateKeyToAddress` (unused).
6. `data.go` — data loading: reads `data/hash160s.json` as the primary source; falls back to `data/wallets.json` (address strings) but conversion is not implemented.
7. `models.go` — JSON structs (`WalletData`, `RangeData`, `Range`, `Hash160Data`).
8. `colors.go` — ANSI terminal color constants.

### Data files (`data/`)

- `hash160s.json` — array of hex-encoded Hash160 values, one per wallet (preferred input)
- `wallets.json` — array of Bitcoin mainnet addresses (used only if `hash160s.json` is absent; conversion path is a stub)
- `ranges.json` — array of `{min, max, status}` objects with 0x-prefixed hex bounds; index aligns 1-to-1 with `hash160s.json`

### `temp/hash160_generator.go`

A standalone utility (its own `main` package) that converts `data/wallets.json` → `data/hash160s.json`. Run it from the `temp/` directory; it resolves paths relative to its parent directory.

### Search algorithm (GPU)

The whole hot loop runs on the GPU (~650 Mkeys/s on an RTX 3050). The design is
the VanitySearch-style "group + symmetry" scheme:

- **Walks.** `searchForPrivateKey` (`search.go`) starts `targetWalks` (64K, rounded
  to `gpuSearchWalkMultiple()`) independent walks — one per kernel thread. Each
  gets a uniformly-random start in `[minKey, maxKey]`. Random starts make each run
  sample different regions of the range.
- **Groups.** Each round, a walk covers the `GRP_KEYS = 2*GRP_HALF+1` keys *centred*
  on its current point: the centre itself plus `P ± iG` for `i = 1..GRP_HALF`. It
  then jumps the centre forward by `GRP_KEYS`, so successive rounds tile the key
  space contiguously — no gaps, no overlap.
- **One inversion per round.** The whole round costs a single `fe_inv`, for two
  compounding reasons:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lmajowka/cacagpu](https://github.com/lmajowka/cacagpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
