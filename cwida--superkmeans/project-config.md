---
trigger: always_on
description: Working notes for the SuperKMeans repo.
---

# AGENTS.md

Working notes for the SuperKMeans repo.

## What it is

Fast k-means for high-dim vector embeddings, on `float32` **or quantized** vectors (`sq8`,
`lvq4`, `rabitq`) — you pass `float32`, it quantizes internally. Faster than FAISS /
scikit-learn at equal quality (~1M×1536 in seconds). Header-only **C++17** + **Python
bindings**; CPUs (ARM + x86) and GPUs. Core: `SuperKMeans::Train()` in
[`include/superkmeans/superkmeans.h`](include/superkmeans/superkmeans.h).
Paper: https://arxiv.org/pdf/2603.20009.

## Core idea: progressive pruning + PDX

Prunes centroids during the **assignment** step by interleaving a GEMM on the front `d'` dims
with **progressive**-pruning kernels every 64 dims in the trailing ones. Unlike Elkan's (a 1/0
decision), pruning is progressive — it keeps trying to prune while walking the trailing dims.
Vectors use a **hybrid PDX layout** (block-column-major, split every 64 dims) to make this
efficient. Papers: **ADSampling** https://dl.acm.org/doi/pdf/10.1145/3589282 (almost-lossless,
< 0.005 recall loss); **PDX** https://dl.acm.org/doi/pdf/10.1145/3725333.

## Where things live

| Path | What |
| --- | --- |
| `include/superkmeans/superkmeans.h` | Core — `Train()`, assign family, pruning loop |
| `include/superkmeans/hierarchical_superkmeans.h` | Hierarchical variant (use for n > 100K) |
| `include/superkmeans/common.h` | Shared `constexpr` constants + macros/pragmas |
| `include/superkmeans/profiler.h` | `SKM_PROFILE_SCOPE` timing |
| `include/superkmeans/distance_computers/` | **All** SIMD distance kernels |
| `include/superkmeans/quantizers/` | `f32`/`sq8`/`lvq4`/`rabitq` + `quantizer.h`, `sq_common.h` |
| `include/superkmeans/pdx/` | PDX layout + `utils.h` |
| `benchmarks/` | Benchmarks; base example `ad_hoc_superkmeans.cpp` |
| `examples/` | `simple_clustering.{cpp,py}`, `hdf5_clustering.py` |
| `python/` | Bindings (`bindings/bindings.cpp`, `superkmeans/__init__.py`); `python/README.md` |
| `tests/` | GoogleTest C++; `python/tests/` for bindings |

Docs: `README.md`, `INSTALL.md`, `BENCHMARKING.md`, `CONTRIBUTING.md`, `python/README.md`.

## Verification gate (definition of done)

**A change isn't done until all pass. Run in the FOREGROUND — never background these.**

1. **Hierarchical parity (first, before building)** — if the change touched `superkmeans.h`,
   evaluate whether it must also be applied to `hierarchical_superkmeans.h`.
   `HierarchicalSuperKMeans` inherits `SuperKMeans` (ctor + shared members propagate via
   delegation) but **overrides `Train` and the clustering pipeline** — so edits to the
   training/pipeline path do NOT automatically carry over. Apply/verify there too.
2. **Format** — `./scripts/format.sh`, then `./scripts/format_check.sh` clean.
3. **Build** — `cmake . -DSKMEANS_COMPILE_TESTS=ON && make -j$(nproc) tests`, no errors.
4. **C++ tests** — `ctest --output-on-failure` all pass (a few parametrized cases skip by design).
5. **Lint** — `./scripts/tidy_check.sh`: no `.clang-tidy` warnings from `include/superkmeans/`.
6. **Python** — `venv/bin/pip install .` (builds the bindings), then `venv/bin/pytest python/tests/`.
7. **Examples** — `make examples`, then run all of them to completion: the C++ ones
   (`./examples/{simple,hierarchical,quantized}_clustering.out <n> <d> <k>`, e.g. `100000 128 1000`)
   and the Python ones (`venv/bin/python3 examples/{simple,quantized}_clustering.py`;
   `hdf5_clustering.py` too if an `.hdf5` dataset is at hand — they live outside the repo).
8. **AGENTS.md accurate** — if the change invalidated anything here (paths, commands, contracts,
   thresholds, gotchas, SIMD org, style), update this file in the same change.

New feature ⇒ ship a unit test with it (C++ in `tests/`, Python in `python/tests/` if exposed).

## Build & run (beyond the gate)

Header-only; consumers link the `superkmeans` target.
```bash
cmake . && make examples          # examples on by default; ./examples/simple_clustering.out <n> <d> <k>
cmake . -DSKMEANS_COMPILE_BENCHMARKS=ON -DFAISS_OPT_LEVEL="avx512" && make benchmarks
./benchmarks/ad_hoc_superkmeans.out <dataset_id>   # base example + profiling logs
```
Knobs: `-DSKMEANS_MARCH` (default `native`), `-DBLAS_LIBRARIES` (a good BLAS is critical —
distro/apt OpenBLAS is slow, build from source). See INSTALL.md.

## Code style

- **Naming**: `PascalCase` functions/classes/structs; `snake_case` variables/members;
  `UPPER_SNAKE_CASE` constants; namespaces lowercase.
- Follow `.clang-format` / `.clang-tidy`.
- **Memory**: RAII, no raw `new`/`delete`. Buffers that don't need zero-init → `new T[]` in a
  `unique_ptr`, **not** `std::vector`/`resize()`.
- Constants/magic numbers → `include/superkmeans/common.h` as `constexpr`.
- Keep comments simple. TODOs: `TODO(@<github_user>, <priority>): <summary>`.
- **Reuse before writing**: check `common.h`, `quantizers/sq_common.h`, `pdx/utils.h` first.

## Performance

Performance-critical — weigh every copy/allocation.
- **SIMD is centralized.** Distance kernels exist for **NEON / AVX2 / AVX512 / scalar**,
  dispatched at compile time and tagged by `Quantization` scheme (`f32`/`sq8`/`sq4`/`rabitq`).
  **All** SIMD lives in `distance_computers/` — don't scatter it elsewhere; keep all backends in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cwida/SuperKMeans](https://github.com/cwida/SuperKMeans) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
