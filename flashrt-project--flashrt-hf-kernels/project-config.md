---
trigger: always_on
description: This repository packages selected FlashRT kernels for Hugging Face Kernel Hub.
---

# Agent Instructions

This repository packages selected FlashRT kernels for Hugging Face Kernel Hub.
Keep changes scoped to packaging, bindings, tests, benchmarks, and copied kernel
source needed by one package.

## Global Rules

- FlashRT upstream source lives in `../official/FlashRT`.
- Do not modify FlashRT upstream from this repository unless explicitly asked.
- Do not expose FlashRT internal `uintptr_t` or raw stream APIs as public Hub
  APIs. Public functions should accept and return `torch.Tensor` objects.
- Native ops must be registered with `TORCH_LIBRARY_EXPAND(TORCH_EXTENSION_NAME, ops)`.
- Python wrappers must import `ops` from `._ops`.
- If Python code defines Torch custom ops, prefix op names with
  `add_op_namespace_prefix` from `._ops`.
- Package names and public functions should be generic. Avoid names such as
  `qwen`, `pi05`, `groot`, or `motus` unless the package is explicitly a
  model-specific compatibility layer.
- Do not commit build outputs, `result/`, `build/`, generated wheels, or Hub
  upload artifacts.
- Keep dependency declarations in `build.toml`; do not depend on FlashRT local
  `third_party/` paths.
- If CUTLASS is required, use a dependency accepted by the package's pinned
  upstream builder revision (for example `cutlass_4_0` or `cutlass_4_5`) and
  include only package-local overrides.
- Keep public package docs clean. Put local planning notes in `internal-docs/`
  and FlashRT-dependent tests in `internal-tests/`.

## Package Promotion Criteria

A package may be promoted from draft to buildable when:

- `build.toml.draft` has been renamed to `build.toml`.
- `torch-ext/torch_binding.cpp` and headers exist.
- All source files listed in `build.toml` exist.
- Correctness tests cover dtype, shape, device, and stride expectations.
- Benchmarks include at least one generic shape set and one FlashRT-real shape
  set.
- Hub-compatible tests do not depend on `../official/FlashRT`; FlashRT parity
  checks live under `internal-tests/`.
- `kernel-builder build <package>` succeeds locally.
- `kernel-builder check-abi <package>` succeeds for produced native modules.

## Release / HF Jobs Workflow

Before pushing a package change that should rebuild Hub artifacts:

1. Identify every package with changed CUDA/C++ bindings, Python wrappers,
   tests, benchmarks, `build.toml`, `flake.nix`, or `flake.lock`.
2. Run source correctness for those packages with the strictest available
   shape mode. Use package tests rather than model demos as the package gate.
3. Run representative benchmarks for changed performance-sensitive APIs and
   record shape, dtype, tolerance, and hardware in package docs or internal
   notes.
4. Run `check-config` from each changed package directory using the exact
   upstream builder revision in that package's `flake.lock`. A stale local
   builder that rejects a newly supported dependency is not authoritative.
5. For every `[torch-noarch]` package, reproduce the builder's final copied
   layout and import it before using HF Jobs:
   `python scripts/validate_noarch_layout.py <package> --backend cuda` (and
   `--backend rocm` when applicable). This is a mandatory paid-build gate. It
   catches sibling Python packages and other files that exist under
   `torch-ext/` but are omitted because the noarch builder copies only
   `torch-ext/<general.name as module>/`.
6. Keep `build/`, `result/`, `dist/`, wheels, `__pycache__/`, and
   `internal-tests/` outputs untracked. `scripts/prebuild_check.py` is expected
   to fail until ignored local build artifacts are removed or the check is run
   in a clean clone.
7. Use `.github/workflows/build-kernels-hf-jobs.yml` for release packaging and
   upload. Add any newly changed package to the workflow path filters,
   dispatch options, and normal matrix before relying on push-triggered builds.
   These lists must match except for packages such as FA2 with a dedicated job.
8. HF Jobs must run under the `liangsu9988` HF account/namespace. Before
   pushing release-triggering package changes, confirm the repository secret
   `HF_TOKEN` is a valid token for `liangsu9988` or another token with both HF
   Jobs access and `flashrt/<package>` kernel publishing rights. If the token is
   missing, expired, or belongs to the wrong account, the workflow fails before
   compilation with `HTTP 401: {"error":"Invalid username or password."}`.
9. After HF Jobs uploads artifacts, read `general.version` from `build.toml`
   and verify by loading through `get_kernel("flashrt/<package>", version=N,
   trust_remote_code=True)` in a matching PyTorch/CUDA environment. Rerun
   installed-artifact correctness; never assume every package is v1.
10. Run `.github/workflows/mirror-kernels-legacy-model.yml` for every new or
   rebuilt package, then verify both the Kernel Hub `v1` ref and the legacy
   model-repo `v1` ref. A successful source upload without compiled build
   variants is not a release.
11. Use the current upstream builder matrix for normal releases. Do not claim
    compatibility with retired Torch versions by renaming artifact directories.
    `torch.stable-abi` is valid only after `kernel-builder check-abi
    --torch-stable-abi` passes and the binding uses only stable ABI headers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flashrt-project/FlashRT-HF-kernels](https://github.com/flashrt-project/FlashRT-HF-kernels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
