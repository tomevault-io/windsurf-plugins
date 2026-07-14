---
trigger: always_on
description: RingKernel is a GPU-native persistent actor model framework for Rust, focused exclusively on NVIDIA CUDA. It enables GPU-accelerated actor systems with persistent kernels, lock-free message passing, and hybrid logical clocks (HLC) for causal ordering. Proven on H100 (8,698x speedup vs `cuLaunchKernel`) and on 2× H100 NVL (8.7× NVLink P2P migration at 16 MiB; 258 GB/s sustained K2K bandwidth, ~81% of NV12 peak).
---

# CLAUDE.md

## Project Overview

RingKernel is a GPU-native persistent actor model framework for Rust, focused exclusively on NVIDIA CUDA. It enables GPU-accelerated actor systems with persistent kernels, lock-free message passing, and hybrid logical clocks (HLC) for causal ordering. Proven on H100 (8,698x speedup vs `cuLaunchKernel`) and on 2× H100 NVL (8.7× NVLink P2P migration at 16 MiB; 258 GB/s sustained K2K bandwidth, ~81% of NV12 peak).

## Release Status

- **v1.0.0** (shipped): single-GPU persistent actor framework, H100-verified.
- **v1.1.0** (shipped, 2026-04-20): multi-GPU runtime (real `cuCtxEnablePeerAccess` + `cuMemcpyPeerAsync` on 2× H100 NVL), VynGraph NSAI integration points (PROV-O, multi-tenant K2K, hot rule reload, introspection streaming), 6/6 TLA+ specs pass under TLC with no counterexamples. See [`docs/benchmarks/v1.1-2x-h100-results.md`](docs/benchmarks/v1.1-2x-h100-results.md) and [`docs/verification/v1.1-tlc-report.md`](docs/verification/v1.1-tlc-report.md).
- **v1.2 groundwork** on `main` (not yet tagged): see section below.

## v1.2 Groundwork (on `main`, not yet released)

The items below landed after v1.1.0 and are additive / opt-in. They are the v1.2 roadmap goals implementable on 2× H100 NVL without B200 silicon. See `[Unreleased]` in [CHANGELOG.md](CHANGELOG.md) for full detail.

- **Hierarchical work stealing** — intra-cluster DSMEM (`cluster_dsmem_work_steal`) and cross-cluster HBM (`grid_hbm_work_steal`) kernels complete the block → cluster → grid stealing hierarchy that started with the v1.1 intra-block `warp_work_steal`.
- **NVSHMEM symmetric-heap bindings** — opt-in `nvshmem` Cargo feature on `ringkernel-cuda`; `multi_gpu::nvshmem::NvshmemHeap` RAII wrapper over the NVSHMEM host ABI (`attach` / `malloc` / `put` / `get` / `barrier_all` / `fence` / `my_pe` / `n_pes`). Bootstrap (MPI / `nvshmrun` / unique-ID) is the caller's responsibility.
- **Blackwell / sm_100 capability queries** — `GpuArchitecture::supports_{cluster_launch_control,fp8,fp6,fp4,nvlink5,tee}`; `from_compute_capability` routes all 10.x / 11.x / 12.x to the Blackwell preset (covers datacenter sm_100/101/103, consumer sm_120, and DGX Spark GB10 sm_121). Rubin has no announced CC yet — callers select `rubin()` explicitly. Codegen stubs compile; runtime paths wait for B200 hardware.
- **Post-Hopper codegen scalar types** — `ScalarType::BF16`, `FP8E4M3`, `FP8E5M2`, `FP6E3M2`, `FP6E2M3`, `FP4E2M1` with per-type `min_compute_capability()`. CUDA / MSL / WGSL lowerings wired; unsupported targets reject at codegen time.
- **SpscQueue hot-path** — `head` / `tail` / producer stats / consumer stats each on their own 128-byte line (was: single line, full cross-core invalidation per op). `update_max_depth` uses `AtomicU64::fetch_max` instead of a CAS loop.
- **Delta checkpoints** — `Checkpoint::delta_from(base, new)`, `applied_with_delta(base, delta)`, and `content_digest()` for ordered identity-based diffs; recorded parent digest catches wrong-base application.
- **HBM tier direct K2K measurement** — `cluster_hbm_k2k` kernel replaces the snapshot/restart proxy used in the initial v1.1 run; Exp 1 now measures all three K2K tiers (SMEM, DSMEM, HBM) directly.
- **Multi-GPU K2K sustained bandwidth** — `paper_multi_gpu_k2k_bw` micro-bench (paper Addendum 6b): 258 GB/s @ 16 MiB on 2× H100 NVL.
- **Workspace dep consolidation** — 8 crates migrated from `version + path` to `{ workspace = true }`; root `[workspace.dependencies]` has a `ringkernel` facade entry.

## Build Commands

```bash
cargo build --workspace                          # Build entire workspace
cargo build --workspace --features cuda           # With CUDA backend
cargo test --workspace                            # Run all tests
cargo test -p ringkernel-core                     # Single crate tests
cargo test -p ringkernel-core test_name           # Single test
cargo test -p ringkernel-cuda --test gpu_execution_verify  # CUDA GPU tests (requires NVIDIA GPU)
cargo test -p ringkernel-ecosystem --features "persistent,actix,tower,axum,grpc"
cargo bench --package ringkernel

# Examples
cargo run -p ringkernel --example basic_hello_kernel
cargo run -p ringkernel --example kernel_to_kernel
cargo run -p ringkernel --example global_kernel         # CUDA codegen
cargo run -p ringkernel --example stencil_kernel
cargo run -p ringkernel --example ring_kernel_codegen
cargo run -p ringkernel --example educational_modes
cargo run -p ringkernel --example enterprise_runtime
cargo run -p ringkernel --example pagerank_reduction --features cuda

# Applications
cargo run -p ringkernel-txmon --release --features cuda-codegen
cargo run -p ringkernel-txmon --bin txmon-benchmark --release --features cuda-codegen
cargo run -p ringkernel-procint --release
cargo run -p ringkernel-procint --bin procint-benchmark --release
cargo run -p ringkernel-ecosystem --example axum_persistent_api --features "axum,persistent"

# CLI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mivertowski/RustCompute](https://github.com/mivertowski/RustCompute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
