---
trigger: always_on
description: Repository guidance for coding agents and contributors. Follow more specific directory instructions
---

# AGENTS.md

Repository guidance for coding agents and contributors. Follow more specific directory instructions
where present. Keep this file current when the paths, commands, or contracts below change.

## Project and code map

RaBitQ-Library is a C++17 library with Python bindings (`rabitqlib`) for compact vector
quantization and approximate nearest-neighbor search. It provides one-bit and multi-bit RaBitQ
encoding and IVF, HNSW, and SymphonyQG indexes. It targets x86-64 with runtime AVX2/AVX-512 dispatch.
RaBitQ rotates vectors and quantizes residuals relative to centroids; compact codes and correction
factors estimate L2 distance or inner product.

| Path | Responsibility |
| --- | --- |
| `include/rabitqlib/quantization/` | Encoding, packing, reconstruction, and byte layouts |
| `include/rabitqlib/fastscan/` | FastScan interfaces and high-accuracy scanning |
| `include/rabitqlib/index/{ivf,hnsw,symqg}/` | Index construction, persistence, and search |
| `include/rabitqlib/index/{query,estimator}.hpp` | Query state and distance estimation |
| `include/rabitqlib/simd/`, `src/simd/` | Kernel declarations, implementations, and dispatch |
| `src/index/` | Compiled HNSW search kernels |
| `src/utils/cpu_features.cpp` | Generic x86 feature detection |
| `include/rabitqlib/utils/` | Rotation, allocation, buffers, I/O, and helpers |
| `python_bindings/` | pybind11 extension and index wrappers |
| `sample/cpp/`, `sample/python/` | Usage examples |
| `tests/unit/`, `tests/integration/`, `tests/python/` | C++ and Python tests |
| `docs/docs/` | MkDocs site sources |

See [README.md](README.md) for usage, [CONTRIBUTING.md](CONTRIBUTING.md) for tool setup and
[implementation recipes](CONTRIBUTING.md#implementation-recipes), and
[tests/README.md](tests/README.md) for build and test commands.

## Required working boundaries

- Inspect relevant code and tests first. For multi-step work, state brief, verifiable success
  criteria. Make the minimum complete change and preserve existing user edits.
- Proceed with scoped local edits and verification. Ask before adding or upgrading production
  dependencies, breaking public APIs or persisted formats, deleting user data, or publishing
  changes unless the user has already authorized that action. Surface material ambiguity before
  choosing behavior or architecture.
- Do not commit, push, create branches, or rewrite Git history unless explicitly requested.
- Avoid unrelated cleanup and generated-file edits unless the project workflow requires them.
  Never expose credentials or private data in code, logs, or responses.
- Reuse the existing project Python environment for builds, installs, and tests. Do not use system
  or user-global package installation or create/replace an environment without authorization.
  Keep machine-specific interpreter paths and environment names in personal agent configuration.
- Report what changed, the checks actually run, and any failures or unavailable checks. Do not
  claim completion while an in-scope verification failure has a safe, available fix.

## Verification by change type

Run the smallest relevant check first, then broaden for shared contracts and regression risk.
Apply all matching rows; these are conditional requirements, not a checklist for every edit.
Run tests in the foreground. Use the existing project interpreter wherever commands say `python`.
Tool versions and installation instructions are maintained in [CONTRIBUTING.md](CONTRIBUTING.md).

| Change | Required verification |
| --- | --- |
| Repository Markdown, including this file | Review accuracy, local links, and `git diff --check`; no code build required |
| MkDocs content, configuration, or dependencies | Above, plus `python -m mkdocs build --strict --config-file docs/mkdocs.yml` |
| First-party C++ | `./scripts/check-format.sh` and affected build/tests; focused clang-tidy checks are sufficient during iteration, with the full `./scripts/check-tidy.sh build-tidy` required before merging |
| Python sources | `./scripts/check-python.sh` and affected Python tests via `python -m pytest` |
| Bindings or Python-visible C++ behavior | C++ and Python checks above; rebuild/install the package in the existing environment before testing, then verify the imported package/extension paths and that the extension includes the current changes |
| SIMD, metrics, packing, layouts, or index formats | Before building, audit every backend and consumer sharing the contract; build and run relevant reference/compatibility tests with `RABITQ_ENABLE_NATIVE_OPTIMIZATION=OFF` |
| Hot-path algorithm/allocation changes or performance claims | Correctness tests first, then a representative benchmark reporting dataset, CPU, compiler, ISA, threads, latency/throughput, and quality; a correctness fix may omit benchmarks with a documented justification, without making an unmeasured performance claim |
| Examples | Build or run the affected example |
| Shell scripts | ShellCheck on affected scripts; see contributor instructions |
| Build or packaging configuration | Exercise the affected build/package workflow and its tests |

Use the C++ configure/build/CTest commands in [tests/README.md](tests/README.md#quick-start).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VectorDB-NTU/RaBitQ-Library](https://github.com/VectorDB-NTU/RaBitQ-Library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
