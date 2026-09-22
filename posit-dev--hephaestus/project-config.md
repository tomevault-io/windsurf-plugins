---
trigger: always_on
description: Repo-level orientation for working in `hephaestus`. Architecture, module map, and per-module specifics live under `src/CLAUDE.md` and the per-folder `CLAUDE.md` files below it.
---

# CLAUDE.md

Repo-level orientation for working in `hephaestus`. Architecture, module map, and per-module specifics live under `src/CLAUDE.md` and the per-folder `CLAUDE.md` files below it.

## Project

`hephaestus` is a 2D scene renderer for data visualization. The crate exposes a backend-agnostic scene API, two Vello backends over wgpu — Vello Classic (GPU compute) and Vello Hybrid (sparse strips: path processing on the CPU, a render pipeline on the GPU) — and two vector backends that emit markup rather than pixels: SVG, aimed at editable output, and PDF, aimed at a fixed artifact with its fonts embedded. The one future planned backend is Blend2D (CPU raster). Performance for interactive / real-time updates on dense plots is the design driver. WASM must work.

The crate ships two API levels in the same source tree: a low-level scene API (`SceneBuilder` + primitives + layout) and a high-level plot API (`plot::*` — geoms, scales, and the `PlotComposition` orchestrator) built on top of it. See `src/CLAUDE.md` for the split and the rules that govern it.

## Commands

```sh
cargo build                                              # default features (vello + png)
cargo build --no-default-features                        # core types & traits only — no wgpu pulled in
cargo check --target wasm32-unknown-unknown             # wasm is a supported target; catches GL-backend and dep regressions
cargo clippy --target wasm32-unknown-unknown --no-default-features --features webgl,document-read -- -D warnings  # the WebGL2 build: no wgpu at all
cargo build --no-default-features --features vello,png   # explicit feature combination
cargo build --no-default-features --features vello-hybrid,png  # sparse strips instead of compute shaders
cargo build --features window                            # adds winit + the presentation surface
cargo +1.86 check --no-default-features --features document-write --ignore-rust-version  # renderer-free writer on the oldest supported rustc

cargo test                                               # all tests
cargo test --test smoke                                  # the GPU smoke test (requires a working wgpu adapter)
cargo test --test pick_index                             # hit testing; needs no features at all
cargo test --test image_geom                             # raster images through PlotComposition
cargo test --no-default-features --features vello-hybrid --test hybrid  # the sparse-strips backend, end to end
cargo test --test window_blit                            # the window presentation blit, headless
cargo check --no-default-features --features window,vello-hybrid,png  # presentation with no compute-shader backend
cargo test --features document --test document_roundtrip # plot documents: reflow at unseen sizes
cargo test --no-default-features --features document,svg --test document_svg  # document in, SVG out: the wasm SVG client's pipeline
cargo test --features svg --test svg                     # the vector backend, with and without a codec
cargo test --features svg,png --test svg
cargo test --features pdf --test pdf                     # the fixed vector backend
cargo test --no-default-features --features pdf,png --test pdf  # …and its one use for a codec: bitmap color glyphs

cargo clippy --all-features --all-targets -- -D warnings # treat warnings as errors
cargo fmt                                                # rustfmt; always run before declaring a task done

cargo run --example hello                                # renders examples/hello.png — visual sanity check
cargo run --example svg_export --features svg            # one plot as both SVG and PNG, for side-by-side review
cargo run --example pdf_export --features pdf,vello,png  # the same plot as both PDF and PNG
cargo check --no-default-features --features document-read,svg  # renderer-free: document in, SVG out
cargo check --no-default-features --features document-read,pdf  # …and document in, PDF out
cargo run --example image_formats --features jpeg,tiff,webp  # all four raster writers
cargo run --example image_geom                           # raster images placed in a panel, and in markdown
cargo run --example document_placeholder --features vello-hybrid,document-read,png  # the static picture a page shows while the client boots
cargo run --example document_svg --no-default-features --features document-read,svg  # document in, SVG out, no renderer in the build
cargo run --example window --features window             # live window: resize + hover picking
cargo run --release --example window --features window -- 100000  # same scene at N points
cargo run --release --example window --features window,vello-hybrid -- 200000 hybrid  # sparse strips: no draw cap
cargo run --release --example backend_perf --features vello,vello-hybrid,png -- 100000  # per-frame cost, both backends
```

The wasm render client is a separate workspace, so it builds on its own:

```sh
cargo clippy --target wasm32-unknown-unknown --no-default-features \
  --features webgl,document-read -- -D warnings          # the client's default config
cargo clippy --target wasm32-unknown-unknown --no-default-features \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [posit-dev/hephaestus](https://github.com/posit-dev/hephaestus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
