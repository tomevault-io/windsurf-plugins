---
trigger: always_on
description: Guidance for AI coding agents working in this repository. The reader is assumed
---

# AGENTS.md

Guidance for AI coding agents working in this repository. The reader is assumed
to know nothing about the project. Read this file fully before editing.

## Project overview

Ember is a from-scratch **C inference server for DeepSeek-V4-Flash on AMD Strix
Halo (gfx1151)**. It is
**ds4/Dwarfstar's server architecture rewritten clean in C, driving lucebox's
tuned HIP kernels** through a stable C ABI.

The load-bearing decision: the GPU kernels (attention, 256-expert MoE, ROCMFP
quant decode, DSpark speculative decode, KV snapshot/restore)
and the tokenizer (a `joyai-llm` pre-tokenizer variant that must be byte-exact)
are **reused** via a vendored engine — they are the entire performance advantage
and represent person-years of gfx1151-specific tuning. Everything above the
forward pass is **rewritten fresh in C** in this repo. This is a *server rewrite
with a kernel bridge*, not a kernel rewrite.

The published full-ROCMFP affine fp2 model (85.3 GiB, 2.58 bpw) meets
the Strix-Halo reference benchmarks (~248–253 tok/s sparse prefill, ~32 tok/s
decode with DSpark). See `README.md` for installation and first use, and
`ARCHITECTURE.md` for the layering rationale.

Primary documentation to consult, in order:

- `README.md` — prerequisites, container quick start, first request, and
  development commands.
- `ARCHITECTURE.md` — the layering and why the server was rewritten but the
  kernels reused.
- `docs/continuous-batching.md` — resident-session batching design.
- `docs/quant-quality-reports.md` — quant evaluation workflow and release gates.
- `CLAUDE.md` — a parallel guidance file with overlapping content; keep both
  files consistent when you change build/test/convention facts.

## Repository layout

```
src/server/    HTTP/1.1 (http.c), SSE streaming (sse.c), chat completions
               (chat_api.c), protocol adapters (api_adapters.c: OpenAI Chat,
               Responses, legacy Completions, Anthropic Messages), context
               compaction (compaction.c), background gating (background_gate.c),
               and main.c — the request-pipeline hub (run_chat).
src/model/     DSML chat template (chat_template.c), tool-call parsing
               (tool_parser.c, dsml_decode.c), exact-token tool replay
               (tool_memory.c), cross-restart continuation (continuation.c),
               KV prefix cache (kv_cache.c), GGUF metadata (gguf.c), model
               cards (model_card.c).
src/common/    buf.h (the universal growable buffer), json.c/json.h/json_util.h,
               utf8.h.
src/backend/   ember_backend.h — THE stable C ABI. Two implementations:
               backend_stub.c (deterministic, GPU-free; drives every test) and
               backend_dflash.cc (extern "C" shim over the vendored engine).
engine/        Vendored fork of lucebox: ggml fork with ROCMFP kernels
               (engine/ggml), the DeepSeek4 backend (engine/dflash/deepseek4),
               batching machinery (engine/dflash/common), HIP compat shims.
               Provenance pinned in engine/VENDOR.md.
test/          Plain C/C++ test binaries (one main() each) plus Python
               server-level and quant-pipeline tests.
scripts/       build.sh (ROCm container build), diagnostics, and the quant
               quality pipeline (*.py, stdlib-only).
share/         model_cards/ (per-model defaults sidecar JSON + _schema.json),
               quant_eval/ (eval fixtures).
reports/       Generated quant quality reports (Markdown/JSON/CSV/SVG).
docker/        Multi-stage Dockerfile: full-ROCm `dev` toolchain and minimal
               dependency-closure `release` image (published through GHCR).
tools/         segvtrace.c — crash-backtrace shim LD_PRELOAD'd in production.
docs/          Design/audit documents listed above.
```

There are no `pyproject.toml`/`package.json`/`Cargo.toml` files: the build is
pure CMake, and the Python scripts use only the standard library.

## Build and test

Two build configurations. **Almost all work happens in the first one.**

```bash
# GPU-free: server + stub backend + full test gauntlet. Builds on any host.
cmake -S . -B build && cmake --build build && ctest --test-dir build

# Real backend (ROCm/HIP; MUST run in the container — no HIP toolchain on host)
docker build --target dev -f docker/Dockerfile \
  -t ember-rocm:7.14-dev .                                # once
scripts/build.sh                                          # -> build-rocm/ember-dflash
```

- `build/` is usually already configured, so `cmake --build build` is the fast
  inner loop.
- Every test is a plain binary with a `main()`; run it directly for full output
  or via ctest by name (ctest names drop the `test_` prefix):

  ```bash
  ./build/test_sse                       # direct: prints every FAIL line
  ctest --test-dir build -R sse -V       # by ctest name
  ctest --test-dir build --output-on-failure
  ```

- Tests use a hand-rolled `CHECK(cond, msg)` macro with `g_pass`/`g_fail`
  counters — no framework. **Adding a test file requires a new
  `add_executable` + `target_link_libraries(... ember_core m)` + `add_test`
  triple in the root `CMakeLists.txt`.**
- **Two CMake source lists must stay in sync.** `ember_core` (stub build) and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [otheru-ai/ember](https://github.com/otheru-ai/ember) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
