---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`pixal3d.cpp` is a private fork of [`pwilkin/trellis.cpp`](https://github.com/pwilkin/trellis.cpp)
(a GGML-based C++ port of Microsoft TRELLIS.2-4B image→3D). The fork's purpose is to build
**Pixal3D** — a TRELLIS.2-backbone model with multiview, camera-aware conditioning — on top of the
inherited trellis.cpp pipeline, while keeping one common C++ pipeline across CUDA, Vulkan, and
future WebGPU/WASM targets. Read `PIXAL3D.md` first; it is the top-level map to the
Pixal3D-specific docs (`docs/PIXAL3D_ARCHITECTURE.md`, `PIXAL3D_PORTING_PLAN.md`,
`PIXAL3D_OP_SUPPORT_MATRIX.md`, `PIXAL3D_VALIDATION.md`, `PIXAL3D_DESIGN_DECISIONS.md`,
`PIXAL3D_UPSTREAM_POLICY.md`, `PIXAL3D_ROADMAP.md`).

Remotes: `origin` = private dev repo (`raven38/pixal3d.cpp`), `upstream` = read-only
(`pwilkin/trellis.cpp`), used only to inspect/selectively cherry-pick. See
`docs/PIXAL3D_UPSTREAM_POLICY.md` before merging or diverging from upstream — keep merges
infrequent and pin exact upstream/ggml SHAs while Pixal3D parity work is in progress; don't update
ggml and Pixal3D graph logic in the same commit.

Until native Pixal3D parity is reached, most work is porting/extending the inherited
trellis.cpp TRELLIS.2 pipeline (see `docs/PIXAL3D_PORTING_PLAN.md` phases) rather than modifying
Pixal3D-specific code that doesn't exist yet — check which phase is current before assuming
Pixal3D conditioning code (ProjectAttention, projection, NAF, multiview fusion) is present.

## Build

```sh
git submodule update --init --recursive
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DGGML_VULKAN=ON   # or -DGGML_CUDA=ON / -DGGML_HIP=ON
cmake --build build -j
```

macOS/Apple Silicon needs no backend flag — Metal is enabled automatically. WebGPU (native Dawn):
`-DGGML_WEBGPU=ON -DGGML_METAL=OFF -DCMAKE_PREFIX_PATH=<prebuilt Dawn>`; the configure step then
applies `patches/ggml-webgpu/*.patch` to the submodule working tree (`docs/GGML_FORK_DIFF.md`
"Local patches" — never commit the resulting submodule diff). Browser SS path: `scripts/build_wasm_ss.sh`
→ `web/ss/` (Worker + WORKERFS + JSPI; `web/ss/run_playwright.js` drives Chrome). A local, gitignored
`build-baseline-metal/` may exist from the Phase 1 baseline build; don't assume it's current.
The vendored `thirdparty/ggml` (branch `trellis-patches` of `pwilkin/ggml`, v0.15.1+10) already
ships the upstream `ggml-webgpu` backend (`-DGGML_WEBGPU=ON`, needs Dawn natively or emdawnwebgpu under
Emscripten); see `docs/spec/31-webgpu-bringup.md` for its state, gaps and build paths.

Studio (desktop/web frontend, `app/`):

```sh
cd app && npm install
npm run dev          # browser dev server
npm run tauri dev    # desktop shell
npm run build         # tsc --noEmit + production bundle
```

## Tests

Tests are standalone `trellis-test-*` executables (`src/test_*.cpp`), not registered in CTest.
Build and run only the target relevant to your change:

```sh
cmake --build build --target trellis-test-<name> && ./build/trellis-test-<name>
```

Available: `trellis-test-ss-flow`, `trellis-test-shape-flow`, `trellis-test-ss-sample`,
`trellis-test-birefnet`, `trellis-test-preprocess`, `trellis-test-deform`, `trellis-test-ss-dec`,
`trellis-test-slat-shape`, `trellis-test-dinov3`, `trellis-test-sparse-conv`, `trellis-test-bigop`,
`trellis-test-c2s`, `trellis-test-sparse-degenerate`, `trellis-test-shape-dec`, `trellis-test-ss-full`,
plus CUDA-only `trellis-test-fa-mask-overflow` / `trellis-test-fa-bf16-range`. Pixal3D stages:
`trellis-test-proj-grid`, `trellis-test-pixal3d-{cond-ss,cond-slat,cond-tex,ss-flow,ss-sample,slat-flow,slat-sample,shape-decode,tex-decode}`,
plus `trellis-test-pixal3d-real-e2e` (the browser real-input E2E's shared C++ driven natively —
`PIXAL3D_DUMP_FIXTURE=<dir>` writes the Shape-1024 fixture the browser partial E2E consumes)
(`[gpu]` arg: -1 = CPU backend, 0 = the build's GPU device; on WebGPU pass `TRELLIS_NOFA=1` to the
flow tests — the backend has no BF16 FlashAttention — and `dec_gpu=-1` to `ss-sample`, whose SS
decoder needs Conv3D). `trellis-webgpu-smoke` / `web/smoke` are the backend smoke tests.

`trellis-smoke` exercises the broader pipeline. Most neural parity tests need reference tensors
dumped by the matching `tools/ref_*.py` script (run via the `uv` venv — numpy, safetensors, gguf,
pillow, torch, timm) before the C++ test can compare against them. There is no coverage threshold;
when reporting test results, state the backend, hardware, and fixtures used — high-memory CUDA
regressions are explicit manual checks, not automated.

**Never validate only the final mesh/GLB.** When adding or debugging a pipeline stage, compare
intermediate tensors stage-by-stage against the PyTorch reference (image → DINO → projection → MV
fusion → ProjectAttention → every flow block → SS decoder → sparse flow → sparse decoder → mesh),
using identical serialized initial noise across PyTorch/native/WebGPU. This is the project's
validation principle (`PIXAL3D.md`), not optional diligence.

## High-level architecture

```
                    PyTorch Pixal3D
                    golden reference
                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raven38/pixal3d.cpp](https://github.com/raven38/pixal3d.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
