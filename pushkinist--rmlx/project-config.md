---
trigger: always_on
description: Rust-native, single-binary MLX inference + conversion backend for Apple
---

# rMLX — agent guide

Rust-native, single-binary MLX inference + conversion backend for Apple
Silicon. Goal: the fastest fully-featured **native, no-Python** backend for
MLX-format models.

## Local-only machine paths

Paths in this file are **relative on purpose** — it is checked in and public.
Concrete absolute machine paths (the model-snapshot root `RMLX_O_MODELS_ROOT`,
the single-MLX claim file under `/tmp`, and local sibling repos) live in a
**gitignored** `LOCAL.md` at the repo root. Use it as a local resolver; never
copy an absolute path from it into this file, a commit, a report, a log, or
any artifact that leaves the machine.

## What this project is

One `cargo build --release` binary that:

1. Loads any MLX-format model (`safetensors`, `mlx-community` layout) with
   **no Python at runtime**.
2. Serves an **OpenAI-compatible HTTP API** — text, plus image and audio
   input for models that support those modalities.
3. Supports the **widest weight × KV quantization matrix** MLX can express,
   including rotation-based KV families no other MLX server ships
   (TurboQuant, IsoQuant, PlanarQuant, RotorQuant). ParoQuant is supported
   too, on the **weight** side — it is not a KV method (see
   `docs/WEIGHT_QUANTS.md` §7).
4. **Converts** models between quant formats / layouts (re-quantize, KV-quant
   repack) — MLX in, MLX out.
5. Multi-model lifecycle (load on demand, unload on idle), but enforces a
   **single MLX process at a time** (Apple Silicon Metal context is exclusive
   per process).

## Documentation map

Subsystem references live under `docs/`. Read these to understand specific
areas before touching code:

| Doc | Topic |
|---|---|
| [`docs/CLI.md`](docs/CLI.md) | rmlx CLI: subcommands, flags, env vars, claim file |
| [`docs/SERVER.md`](docs/SERVER.md) | HTTP server: OpenAI/Anthropic compat, routes, tool calling, retry envelope |
| [`docs/MODELS.md`](docs/MODELS.md) | Per-architecture model reference (Qwen, Gemma, Laguna, Jina, etc.) |
| [`docs/ADDING_A_MODEL.md`](docs/ADDING_A_MODEL.md) | New-arch integration surface: shared seams + per-arch points + verification ritual |
| [`docs/WEIGHT_QUANTS.md`](docs/WEIGHT_QUANTS.md) | Weight quantization formats (mxfp, affine, TurboQuant, PlanarQuant, ParoQuant) |
| [`docs/KV_QUANT.md`](docs/KV_QUANT.md) | KV-cache quantization variants (K8V4, K8V8, Mixed, Planar, Paged, rot_k) |
| [`docs/KV_CACHE.md`](docs/KV_CACHE.md) | KV cache architecture (block alignment, ring buffer, SWA snapshot, chunked prefill) |
| [`docs/SSD_TIER.md`](docs/SSD_TIER.md) | SSD KV tier (layout_key, ssd_index schema, hydrate, spill, cross-namespace LRU) |
| [`docs/SSD_CANARY.md`](docs/SSD_CANARY.md) | SSD KV cross-restart smoke probe |
| [`docs/PROMPT_CACHE.md`](docs/PROMPT_CACHE.md) | Prompt cache + automatic prefix caching (block hashing, ReusePolicy, prefix index) |
| [`docs/SPECULATIVE.md`](docs/SPECULATIVE.md) | Speculative decoding (MTP, DFlash, Eagle3 drafters; round-loop; accept-rate gates) |
| [`docs/SAMPLING.md`](docs/SAMPLING.md) | Per-token sampling (temperature, top-k/p, penalties, thinking budget, constrained decoding) |
| [`docs/FFI.md`](docs/FFI.md) | rmlx-mlx ↔ mlx-c FFI bridge; MSL kernel surface; unsafe policy |
| [`docs/METRICS_DB.md`](docs/METRICS_DB.md) | Metrics DB: observations / events / bests; ingest, query, export, deltas |
| [`docs/PERF_BASELINE.md`](docs/PERF_BASELINE.md) | Recorded decode-TPS anchors per (model, KV quant) cell |
| [`docs/PROFILING.md`](docs/PROFILING.md) | samply / Instruments flamegraph workflow |
| [`docs/PROJECTS_CONFIG.md`](docs/PROJECTS_CONFIG.md) | Per-project cap defaults via `<RMLX_HOME>/projects.toml` |
| [`docs/TESTING.md`](docs/TESTING.md) | RMLX_TEST_MODEL_* env vars + RMLX_O_MODELS_ROOT for test snapshot resolution |
| [`docs/RELEASING.md`](docs/RELEASING.md) | Release flow: single-source version, `make tag` / `release-package` / `tap-sync`, Homebrew formula + tap, `CHANGELOG.md` |

Subdir `docs/superpowers/` holds process artifacts — not a subsystem reference.

## What this project is not

- Not a GGUF runtime — that is `llama.cpp`'s lane.
- Not training / fine-tune / fuse / lora-merge. Conversion is not training.
- Not a Python tool. Native Rust only.

## Status — where we are going

Target **0.1.0**: a fully functional native MLX backend with broad feature
and quantization coverage. Scope:

- **Text** generation, OpenAI-compatible.
- **Image input** for models that accept it (vision towers).
- **Audio input** for models that accept it.
- **Agent integration** — tool / function calling, multi-turn, the full
  agent-driving surface.
- **Models from the `RMLX_O_MODELS_ROOT` folder** served end-to-end.
- **Maximum quantization coverage** — every weight and KV quant we can
  support, including the rotation-based KV families.
- **Conversion** — quant↔quant and layout repack as a first-class command.

Build a fast, native, no-Python backend. Port from and study the sibling
repos rather than reinventing.

## Test targets

Under `RMLX_O_MODELS_ROOT` (the dev checkout uses `../../O-Models/`; public
users set it via `.env`). At minimum these three families must serve
end-to-end at every change:

| Family | Example snapshot | Arch |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pushkinist/rMLX](https://github.com/Pushkinist/rMLX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
