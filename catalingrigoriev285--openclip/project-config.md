---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

openclip is a cross-platform screen recorder in Rust (edition 2024, Rust 1.85+) with an egui/eframe GUI. It writes standard MP4 or AVI with **no external runtime dependencies**: OpenH264 and LAME are compiled from bundled sources by their crates at build time, hardware H.264/HEVC and AAC encoders come from Windows Media Foundation (part of the OS, Windows only), and the MP4 and AVI muxers are in-house. A C/C++ toolchain is required to build (MSVC on Windows; autotools on macOS/Linux — see README for the full package list). Installing `nasm` before building enables OpenH264's assembly kernels (~2× encoder throughput); without it the build silently falls back to C.

## Commands

```sh
cargo build --release                 # first build compiles OpenH264 + LAME from source (slow)
cargo run --release                   # launch the GUI
cargo test                            # unit tests + tests/mp4_roundtrip.rs + tests/avi_roundtrip.rs
cargo test --test mp4_roundtrip       # just the MP4 round-trip integration test
cargo test --test avi_roundtrip       # just the AVI round-trip integration test
cargo test --lib avc                  # run unit tests matching a name
cargo clippy --all-targets

# Headless examples (useful for testing the pipeline without the GUI)
cargo run --release --example capture_to_mp4 -- 10 out.mp4   # record primary monitor for 10 s
#   flags: --half --mic --no-audio --fx --region X,Y,W,H --window TITLE --pause-at S --resume-at S
#          --codec openh264|h264-hw|h264-sw|hevc|<label substring> --audio mp3|aac|pcm --avi --fps N --quality Q
cargo run --release --example bench_encode -- 1920 1080 5 out.mp4 --codec nvenc   # encoder throughput on synthetic content
cargo run --example list_encoders                            # every Media Foundation encoder + whether it activates
cargo run --example check_update                             # what the updater sees; `-- --install` runs the full self-update on the example exe

# Game capture, without needing a game installed
cargo run --example gfx_sandbox -- --vsync --verify           # a D3D11 app that reports whether the overlay is in its back buffer
#   flags: --resize-after S (exercise ResizeBuffers), --size WxH
cargo run --example inject_test -- --pid <n> --record out.mp4 --seconds 5   # probe, inject, arm, record
#   flags: --exe <window-title-substring> instead of --pid
```

`profile.dev` uses `opt-level = 1` for the crate and `opt-level = 3` for dependencies, so debug builds are usable for real-time capture. In release builds `main.rs` sets `windows_subsystem = "windows"` (no console); use a debug build or `RUST_LOG` via `env_logger` when you need log output (`RUST_LOG=openclip=trace` also traces the Media Foundation event loop).

`.gitignore` excludes `*.mp4`, `*.avi`, `*.h264`, `*.wav`, `*.mp3` — examples and manual tests can write scratch media into the repo root safely.

## Architecture

The crate is a library (`src/lib.rs`) plus a thin binary (`src/main.rs`) that only builds the eframe window and instantiates `openclip::ui::App`. Examples and the integration tests link against the library API, so keep pipeline/codec code out of `ui`.

The root package is **also the workspace root** (`[workspace] members = ["crates/openclip-overlay"]`, `resolver = "3"`), so `[profile.*]` keeps applying to everything and `target/` stays put. `crates/openclip-overlay` is a deliberately dependency-light library (`ab_glyph`, optional `serde`) holding everything the **injected game hook** and openclip itself must agree on, because the hook is mapped into other people's game processes and cannot link egui: `Sprite` (straight-alpha RGBA with `put` / `get` / `fill_pill`), `TextRenderer` (the `ab_glyph` glyph-run loop, `width`, `baseline`, `draw_run` with a per-character colour so one kerned run can be two colours), `INTER_SEMIBOLD`, `Corner` + `Layout` (corner placement and frame-relative sizing), and `FpsOverlay` / `FpsBadge` / `HookState` (the in-game counter: green = armed, red = recording). `src/video/watermark.rs` composes its badge from exactly these, so the two overlays cannot drift apart in metrics or placement; only `blit` (which needs `RawFrame` and its pixel order) stays on the openclip side. The crate installs no logger and allocates nothing at load time — keep it that way.

```
capture backend ──RawFrame──▶ encode thread: scale → I420/NV12 → VideoEncoder → Muxer (MP4 | AVI) ──▶ file
                                   ▲                                                    ▲
cpal (mic / loopback) ──▶ mixer ───┴──▶ AudioEncoder (MP3 | AAC | PCM) ─────────────────┘
```

### `src/i18n.rs` — interface language


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [catalingrigoriev285/openclip](https://github.com/catalingrigoriev285/openclip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
