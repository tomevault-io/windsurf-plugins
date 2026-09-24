---
trigger: always_on
description: Multi-backend inference engine for DeepSeek-V4, MiniMax, GLM, and Qwen checkpoints. One repository,
---

# PocketLLM

Multi-backend inference engine for DeepSeek-V4, MiniMax, GLM, and Qwen checkpoints. One repository,
two engines: a device-agnostic core shared by both, and a per-vendor kernel layer.

`docs/README.md` indexes the documentation — model support status, benchmarking rules, and the
release procedure are documented there.

## Language convention

**All Markdown documents and code comments in this project must be written in English**, unless a
Chinese version is explicitly requested as an additional deliverable.

When a Chinese version is requested, keep it as a separate file (see `README.md` / `README_CN.md`)
rather than mixing languages inside one file.

This applies to commit messages, code comments, docstrings, and all `.md` files.

## Documentation layout

**New documents go into the existing topic directory. Do not add a file at the top level of
`docs/`.** The top level holds the two site entry points and nothing else — `docs/README.md`, which
indexes the directories, and `docs/getting-started.md`, which the `mkdocs.yml` nav pins at that path.

| Directory | What belongs in it |
|---|---|
| `docs/guides/` | Rules and procedures — benchmarking, API, release flow, Ascend platform notes |
| `docs/architecture/` | Design documents, refactor plans, **roadmaps**, engine comparisons |
| `docs/performance/` | Measured results and bottleneck analyses for capability that is live today |
| `docs/models/` | Per-checkpoint guides and the support matrix |
| `docs/migration/` | Breaking-change migration notes |
| `docs/reports/` | Rendered long-form reports |
| `docs/archive/phase2-phase3/` | Completed Phase 2/3 records, kept for measurement context |

Filenames are lowercase `snake_case`. Every directory has an `index.md` listing its documents in a
table, and `docs/README.md` indexes the directories — a new document that is not added to its
directory's `index.md` is unreachable except by guessing a path, so **update the index in the same
commit**. Relative links between directories need the `../` prefix; moving a file means fixing every
inbound reference in the same commit (source comments and test headers link here too, not just other
Markdown).

This is not only a filing convention: `docs/` is the published site and the build runs
`mkdocs build --strict`, so a link that no longer resolves fails the page build rather than just
looking untidy.

## Repository layout

| Path | What it is |
|---|---|
| `cpp_engine/` | C++ engine. `core/` is device-agnostic, `engine/` is the layer model, and `backends/{api,cuda,ascend}/` holds the vendor code. Build and run instructions: `cpp_engine/README.md`. |
| `src/` | Python/PyTorch implementation and kernel library. |
| `pocketllm/` | The installed package: CLI, HTTP server, supervisor. Imports `pocketllm_cpp` when the native engine was built. |
| `tests/` | pytest suite — see **Testing** below. |
| `docs/` | Topic directories — `guides/`, `architecture/`, `performance/`, `models/`, `migration/`, `reports/`, `archive/` — indexed by `docs/README.md`. Also the source of the published site: `mkdocs.yml` points `docs_dir` at it and `.github/workflows/pages.yml` builds it to <https://lvyufeng.github.io/PocketLLM/>. New files go in a topic directory, never at the top level; see **Documentation layout** above. |

Two invariants the layout exists to protect:

- **Kernels stay behind the C ABI.** `cpp_engine/include/cuda_ops.hpp` declares 93 ops, 91 of which
  take `void* stream`, and it includes no CUDA headers at all. `cpp_engine/engine/` likewise
  includes no CUDA headers and contains **zero `<<<` launches** — all 359 of them are under
  `backends/`. Adding a vendor-specific type to an op signature breaks the other vendor's build.
- **Backend selection happens at build time.** Per-hardware tuning is not traded away for
  portability, so do not "unify" a 2080 Ti or 910B specific kernel in the name of sharing code.

Quantized kernel dispatch on the Python side goes through `src/kernels/ops.py`
(`_auto_impl` / `_resolve_impl`), with paired `*_torch` / `*_triton` implementations behind it.

## Hardware and toolchain

Two development machines, one per backend. **Determine which one you are on before concluding
anything about what can be built, run, or measured.** A command that is correct on one is usually
wrong on the other — most visibly, the CUDA path does not exist on the Ascend machine at all.

### x86_64 CUDA machine — 4 x RTX 2080 Ti

- **GPUs**: 4 x RTX 2080 Ti, 22528 MiB each, compute capability **7.5 (Turing / sm_75)**.
  `cpp_engine` defaults to `CMAKE_CUDA_ARCHITECTURES=75`; do not drop sm_75-specific paths.
- **Topology**: `GPU0-GPU1` are PHB (PCIe, same NUMA node); **`GPU2-GPU3` are NV2 (NVLink)**; every
  cross-pair is SYS. NVLink-sensitive work and fair TP2 comparisons must run on physical GPUs
  **2 and 3**.
- **CPU / RAM**: 2 x Xeon E5-2696 v4, 22 cores each (88 hardware threads), 2 NUMA nodes, ~1 TiB RAM.
  GPUs 0-1 sit on NUMA node 0, GPUs 2-3 on node 1.
- **OS / Python**: Ubuntu 22.04.5, x86_64, kernel 5.15. Python 3.10.10 (conda).
- **Compilers**: gcc 11.4.0, cmake 3.26.3 (conda's, first on `PATH`).
- **CUDA**: `nvcc` on `PATH` is **13.0** (`/usr/local/cuda` → 13.0) while `CUDA_HOME` points at

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lvyufeng/PocketLLM](https://github.com/lvyufeng/PocketLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
