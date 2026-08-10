---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**yosh** is a from-scratch, high-throughput local manga/comic reader in Rust (`winit` + `wgpu` + `egui`).
The single defining feature is **zero-hitch page turning and continuous scrolling**: a parallel
decode-ahead pipeline keeps display-ready GPU textures buffered ahead of the read position, so a page
change is just a texture swap. Performance is the point — treat the decode/render hot path as
load-bearing and don't regress seek throughput (target ≈ 83 pages/sec; HQ path measures ~110+ pps).

Cargo workspace, three crates:
- `crates/yosh` — the application (everything below lives here).
- `crates/decode_bench`, `crates/present_bench` — throwaway Phase-0 spikes that validated the
  throughput ceiling before the real build. See `SPIKES_RESULTS.md`. Not part of the app.

## Commands

```sh
cargo run -p yosh -- "<path>" [start_page]            # dev (debug; keeps a console for logs)
cargo run --release -p yosh -- "<path>" [start_page]  # release (perf-representative; GUI-subsystem)
cargo build --release -p yosh                          # build the shipping binary
cargo check -p yosh                                    # fast type-check while iterating
cargo test  -p yosh                                    # unit tests (live in layout.rs: spread/RTL pairing math)
cargo test  -p yosh spread                             # run a subset of tests by name substring (e.g. spread_navigation)
```

- `<path>` = a folder of images, or a `.cbz/.zip`, `.cbr/.rar`, or `.7z/.cb7` archive. No arg → library grid / keys overlay.
- **The default build needs no C toolchain** — all decoders are pure-Rust (`png`, `jpeg-decoder`,
  `image`, `qcms`), TLS for self-update is `ureq` over rustls+**ring** (not aws-lc). **Preserve this.**
  The one exception is AVIF, gated behind an off-by-default feature:
  `cargo build --release -p yosh --features avif` (needs `nasm` + `dav1d`).
- No rustfmt/clippy config, no toolchain pin; edition 2024 (uses let-chains). Standard `cargo fmt` / `cargo clippy`.
- Release builds are GUI-subsystem on Windows (`#![cfg_attr(not(debug_assertions), windows_subsystem="windows")]`),
  so no console on double-click; `main.rs::reattach_console()` rebinds stdio to the parent console when
  launched from a terminal. Debug builds keep a normal console.

## Architecture (the big picture)

### The decode-ahead pipeline (the heart of the app)
- **`pool.rs` — `DecodePool`**: N worker threads. Each worker reads → decodes → downscales → uploads a
  page to its own GPU texture, **entirely off the main thread** (wgpu `Device`/`Queue` are `Send+Sync`,
  so workers call `write_texture` themselves). The main thread only swaps in finished textures.
  - The scheduler rebuilds the **nearest-first** job list on *every* navigation (`set_jobs`), so workers
    always grab the highest-priority page relative to the latest position. `poll()` drains finished pages.
  - `inflight` dedups; jobs already running are not re-queued.
- **`source/mod.rs` — `PageSource` trait** (`Send + Sync`): the abstraction the pool pulls from —
  `read_page(index) -> bytes`, `len`, `name`, `modified`. Implementations: `FolderSource`, `ZipSource`
  (parallel reads), `RarSource` + `SevenzSource` (sequential formats: a reader thread decompresses into
  an in-memory map, then reads are served from there).
- **Decode + HQ downscale** (`decode.rs` + `tone.rs` + `icc.rs`): magic-byte routing to png / jpeg-decoder
  / image. This is subtle and deliberate:
  - **Grayscale path** downscales in **true linear light** (sRGB → 16-bit linear → Catmull-Rom resample →
    re-encode through the Dot Gain 20% curve in `tone.rs`). This is what kills halftone-screentone moiré;
    resampling in gamma/perceptual space does **not**. Don't "simplify" it back to a gamma-space resize.
  - **Color path**: Lanczos3, plus `qcms` ICC → sRGB color management (`icc.rs`), applied *before* resize.
  - Color decodes that are *visually* grayscale are detected (`rgba_is_grayscale`) and routed to the gray path.
  - **Single-resize invariant (load-bearing for quality — do not regress).** A page displayed at ≤ its
    native resolution must be resampled **exactly once**, by the HQ CPU resize above. The GPU then samples
    that texture **1:1** at draw time; there must be no second (bilinear, no-mipmap) downscale at draw, which
    would soften the image and re-introduce screentone moiré. The *only* permitted GPU resample is
    **upscaling** when zoomed past native resolution (magnification — there's no source detail to do a CPU
    resize, and the texture is capped at the source size). The per-page decode target is bounded only by the
    GPU's real `max_texture_dimension_2d` (`decode::MAX_TEX_DIM`, aspect-aware so the width fits too) — **not**
    a smaller fixed cap. A former fixed 3840-px cap silently forced a GPU *upscale* (→ moiré) on any page
    taller than 3840 viewed near native, because the texture couldn't be decoded to the shown size; decoding to
    the display size (up to the GPU limit) keeps the GPU at 1:1 below native. A *transient* GPU resample

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-database/yosh](https://github.com/the-database/yosh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
