---
trigger: always_on
description: Guidance for Claude Code working in this repo. Keep lean: only things not obvious from the code, `--help`, or `git log`.
---

# CLAUDE.md

Guidance for Claude Code working in this repo. Keep lean: only things not obvious from the code, `--help`, or `git log`.

## What mold is

Local AI image/video generation CLI built on [candle](https://github.com/huggingface/candle). Supports FLUX, SD1.5, SDXL, SD3.5, Z-Image, Flux.2 Klein, Qwen-Image, Wuerstchen v2, LTX-Video, and LTX-2. Runs locally on GPU or talks to a remote `mold serve` over HTTP. Single binary, everything feature-gated.

## Commands

```bash
# Nix (preferred)
nix build                   # Build mold (default CUDA/Metal)
nix fmt                     # treefmt (nixfmt + rustfmt)
nix flake check             # CI-equivalent gate

# Cargo — common loops
cargo check                                                                    # type check
cargo clippy --workspace --all-targets -- -D warnings                          # what CI runs
cargo fmt --all -- --check                                                     # what CI runs
cargo test --workspace                                                         # what CI runs
cargo check -p mold-ai --features preview,discord,expand,tui,webp,mp4         # what CI also runs
./scripts/coverage.sh [--html]                                                 # coverage

# Local dev run (MUST prefix with ensure-web-dist so the embedded SPA isn't a stub)
./scripts/ensure-web-dist.sh && cargo run --profile dev-fast -p mold-ai \
  --features metal,preview,expand -- run "a cat"
```

Inside `nix develop` the devshell exposes shortcuts (`build`, `build-release`, `serve`, `mold`, `clippy`, `run-tests`, `coverage`, `fmt`). Run `type <cmd>` to see the underlying invocation.

**CI gates** (`.github/workflows/ci.yml`): `rust` (fmt + check + clippy-deny-warnings + test + feature-combo check), `coverage` (cargo-llvm-cov → Codecov), `docs` (`bun run fmt:check && bun run verify && bun run build` in `website/`). All must pass.

## Crates

```
crates/
├── mold-core/        Shared types, HTTP client, config, manifest, validation, download
├── mold-db/          SQLite (rusqlite, bundled, WAL) — gallery, settings, model_prefs, prompt_history
├── mold-inference/   Candle engines per family (FLUX, SD1.5/XL/3, Z-Image, Flux.2, Qwen-Image, Wuerstchen, LTX-Video, LTX-2)
├── mold-server/      Axum HTTP server (consumed as lib by mold-cli)
├── mold-cli/         The `mold` binary (clap)
├── mold-discord/     Discord bot (poise + serenity), HTTP-only dep on mold-core
└── mold-tui/         Interactive TUI (ratatui)
```

**Directory ≠ package name.** Use these with `-p`:

| Dir | Package |
|---|---|
| `mold-cli/` | `mold-ai` (binary: `mold`) |
| `mold-core/` | `mold-ai-core` |
| `mold-db/` | `mold-ai-db` |
| `mold-inference/` | `mold-ai-inference` |
| `mold-server/` | `mold-ai-server` |
| `mold-discord/` | `mold-ai-discord` |
| `mold-tui/` | `mold-ai-tui` |

**MSRV**: 1.85.

**Feature flags** (`mold-cli`): `cuda`, `metal`, `preview`, `discord`, `expand`, `tui`, `metrics`, `webp`, `mp4`. GPU features forward through to `mold-inference`. H.264 decode is baseline for LTX-2 source ingest; `mp4` only gates AAC mux.

## Non-obvious architectural patterns

Most are in `mold-inference`. When touching engines, these are the rules that matter:

- **Lazy load** — engines load on first `generate()`, not startup. Most hold mmap'd safetensors.
- **Drop-and-reload text encoders** — T5/CLIP/Qwen3 are dropped from GPU after encoding so the transformer has VRAM to denoise, then reloaded next request.
- **Dynamic device placement** — text encoders go to GPU or CPU based on remaining VRAM after the transformer loads (thresholds: `device.rs`).
- **Quantized encoder auto-fallback** — when FP16/BF16 doesn't fit, the largest GGUF variant that fits is auto-selected. Override: `--t5-variant` / `--qwen3-variant` / `--qwen2-variant` or `MOLD_*_VARIANT`.
- **Block-level offloading** (FLUX) — `flux/offload.rs` streams transformer blocks CPU↔GPU one at a time: ~24 GB → 2–4 GB VRAM, 3–5× slower. Auto-enabled under pressure; force with `--offload` / `MOLD_OFFLOAD=1`.
- **LoRA backend is custom** — candle has no LoRA. BF16 path: `LoraBackend` (a `SimpleBackend`) intercepts `vb.get()` during model construction and applies `W' = W + scale·(B @ A)` inline. GGUF path: `gguf_lora_var_builder()` selectively dequantizes affected tensors, merges, re-quantizes. Both work with offloading. See `flux/lora.rs`.
- **LoRA caching** — `LoraDeltaCache` (pre-computed `B @ A · scale` on CPU, ~80–120 MB) survives transformer rebuilds. `LoraFingerprint` on `FluxEngine` skips redundant rebuilds when the same LoRA/scale reappears.
- **Shared tokenizer pool** — `shared_pool.rs`: `Arc<Tokenizer>` keyed by file path, shared across engines via `create_engine_with_pool()`. Saves ~100–150 ms on model swap for FLUX variants.
- **CPU-based noise** — `seeded_randn()` in `engine.rs` generates initial noise on CPU via `StdRng`/ChaCha20, then moves to GPU. This is load-bearing for cross-backend seed determinism (CUDA/Metal/CPU produce identical images).
- **Z-Image has a bespoke quantized transformer** — `zimage/quantized_transformer.rs` lives here (not candle); GGUF tensor naming differs from BF16 (`attention.qkv` vs split Q/K/V, etc.).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [utensils/mold](https://github.com/utensils/mold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
