---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Project Overview

`bevy_tui_texture` renders ratatui terminal UIs as GPU textures inside a
Bevy app — on 2D UI nodes, 3D meshes, or existing glTF screens — with no
CPU readback anywhere in the hot path.

**Version pins**: bevy 0.19 + ratatui 0.30.2 + wgpu 29 (wgpu must exactly
match bevy's pin — bump together; both are re-exported so downstream code
can name matching types).

## Build & Verify

```bash
cargo build                       # library
cargo build --release             # LTO fat, codegen-units=1

# Examples: ALWAYS `cargo run --example`, never the bare target/ binary —
# assets live in examples/assets/ (shared with the wasm build) and resolve
# via an `AssetPlugin { file_path: "examples/assets", .. }` override
# relative to CARGO_MANIFEST_DIR; a bare binary looks next to the
# executable and e.g. shader_mesh renders black.
cargo run --example helloworld          # minimal static terminal
cargo run --example widget_catalog_2d   # 2D UI, mouse hit-testing, CJK, glyphs
cargo run --example widget_catalog_3d   # 3D mesh terminal with interaction
cargo run --example world_terminal      # world-unit screen (TuiRequest::world_quad)
cargo run --example multiple_terminals  # several terminals + Tab focus cycling
cargo run --example form_demo           # form built on TerminalEvent + HitRegions alone
cargo run --example shader_mesh         # ExtendedMaterial CRT shader effects
cargo run --example retro_crt           # full CRT demo: glTF + shader + overlay UI
cargo run --example resize              # Tui::request_resize following the window
cargo run --example transparent_world_quad  # see-through HUD screen
cargo run --example benchmark           # full-frame throughput
cargo run --example benchmark_partial   # BENCH_MODE=static|partial redraw costs

# Tests: inline #[cfg(test)] modules next to the code (no tests/ dir).
# Pure CPU except one GPU-backed test that skips without an adapter.
cargo test

# What CI runs (.github/workflows/ci.yml):
cargo test --all-features
cargo clippy --all-features --all-targets -- -D warnings
cargo doc --no-deps --all-features
cargo check --lib --no-default-features            # + --features 2d / 3d variants
cargo check --target wasm32-unknown-unknown --example wasm_demo
```

Examples log at `RUST_LOG` level via the dev-dependency bevy's `bevy_log`
feature (the lib's own bevy is default-features=false and logs nothing).

## Feature Flags

- `2d` (default) — 2D UI terminals (`TuiUi`, `TuiKind::Ui`)
- `3d` (default) — 3D mesh terminals (`TuiKind::WorldQuad`, `AttachTerminal`, mesh raycasting)
- `keyboard_input` (default) — keyboard event handling
- `mouse_input` (default) — mouse events for 2D UI and 3D mesh terminals;
  touch rides the same path (touch position feeds `CursorPosition`, a tap
  emulates the left button)
- `bold_italic_fonts` (opt-in) — real bold/italic font slots
  (`Fonts::add_bold_fonts`/`add_italic_fonts`/`add_bold_italic_fonts`);
  otherwise bold/italic are faked from the regular font
- `emoji` (opt-in, WIP) — emoji-aware glyph handling (pulls in `unicode-properties`)
- `ascii_fast_shaping` (opt-in) — bypasses rustybuzz for rows that are all
  single ASCII printable bytes (see IMPROVEMENT.md A3). Assumes zero
  x_offset, which most monospace fonts satisfy but isn't guaranteed.
  Silently inert when `bold_italic_fonts` is enabled (that feature makes
  per-cell font selection meaningful; this path assumes one font per row)
- `crossterm-compat` (opt-in, native-only) — `InputEvent::to_crossterm`/
  `from_crossterm` (src/input/crossterm_compat.rs), lossy conversions
  to/from `crossterm::event::Event` for interop with ratatui-ecosystem
  widget crates (e.g. tui-textarea) or a bevy_ratatui adapter. Not
  available on wasm32 — crossterm doesn't build there

`TuiKind` variants gate individually: `Ui` needs `2d`, `WorldQuad` needs
`3d`, `Headless` is always available. One-surface builds work:
`cargo build --no-default-features --features "3d,keyboard_input,mouse_input"`.

## Core Architecture

### Abstraction ladder (src/setup.rs)

1. **`TuiRequest`** (default choice) — declarative: spawn the component
   (plus any `Node`/`Transform`/markers), `materialize_tui_requests`
   creates the texture and inserts the terminal components next frame — no
   render resources in user code. `TuiKind::Ui` (2D), `TuiKind::WorldQuad
   { height }` (world-unit quad, width follows texture aspect),
   `TuiKind::Headless` (a `Tui` with no surface). Fonts arrive as
   `Arc<Fonts>` (`TuiFontSource::Ready`, via `Into`) or through the
   AssetServer (`TuiFontSource::Asset` — the Wasm-safe path; the request
   stays pending until the `.ttf` loads).
2. **`AttachTerminal` + `AttachMaterial`** (feature `3d`) — put a
   (typically headless) `Tui` on an *existing* mesh, e.g. a glTF
   primitive. `attach_terminal_system` re-claims the material every frame
   until the async loader stops overwriting it, then tracks the installed
   handle in `TuiAttached` and goes idle (no archetype churn once settled).
3. **`TerminalTexture::create` + `Tui::from_texture_state`** — manual
   escape hatch (`examples/tui_component.rs`, `examples/shader_mesh.rs`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tt-toe/bevy_tui_texture](https://github.com/tt-toe/bevy_tui_texture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
