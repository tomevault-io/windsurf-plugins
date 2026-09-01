---
trigger: always_on
description: hyprsaver is a Wayland-native screensaver for Hyprland. It renders GLSL fractal shaders on fullscreen wlr-layer-shell overlay surfaces via OpenGL ES (glow). It integrates with hypridle (timeout orchestration) and coexists with hyprlock (lock screen). The two are intentionally separate — Unix philosophy.
---

# CLAUDE.md — hyprsaver

## Project Overview
hyprsaver is a Wayland-native screensaver for Hyprland. It renders GLSL fractal shaders on fullscreen wlr-layer-shell overlay surfaces via OpenGL ES (glow). It integrates with hypridle (timeout orchestration) and coexists with hyprlock (lock screen). The two are intentionally separate — Unix philosophy.

## Architecture
Eleven modules in `src/` (plus `main.rs`):
- `wayland.rs` — Wayland connection, output enumeration, layer-shell surface lifecycle. Uses smithay-client-toolkit. One surface per monitor. Hosts the calloop event loop, calls `CycleManager::tick(now)` each frame, and dispatches `CycleEvent`s to advance shaders/palettes.
- `renderer.rs` — OpenGL via glow. Fullscreen quad, uploads uniforms (time, resolution, palette vectors, speed/zoom scales, alpha fade), calls draw. Doesn't know about Wayland.
- `shaders.rs` — Loads `.frag` files from config dir and built-ins. Handles compilation, hot-reload (notify crate), Shadertoy uniform remapping. Prepends palette function to all shaders. Manages cycle playlists (`set_playlist`, `cycle_next`, `randomize_cycle_start`).
- `palette.rs` — Cosine gradient palettes (Inigo Quilez technique) and LUT palettes. Four vec3 params (a,b,c,d) → 12 floats. PNG LUT loading via `image` crate. CSS gradient stop palettes. `PaletteManager` with crossfade transition state (`begin_transition` / `advance_transition`).
- `config.rs` — TOML config with serde. Every field has a default. Config path: CLI flag → `$XDG_CONFIG_HOME/hypr/hyprsaver.toml` (new) → `$XDG_CONFIG_HOME/hyprsaver/config.toml` (legacy, deprecated) → built-in defaults. Includes `[playlists.<name>]` table sections (unified v0.4.0 format; legacy `[shader_playlists.<name>]` / `[palette_playlists.<name>]` still parsed), cycle interval fields, and the `[render_preview.palettes]` shader→palette override map.
- `cycle.rs` — `CycleManager`: tick-driven scheduler for shader and palette rotation. `tick(&mut self, now: Instant) -> Vec<CycleEvent>` returns an empty vec when nothing changed. `CycleOrder` supports `Random` (shuffle-bag, no consecutive repeats across bag boundaries) and `Sequential`. Single-item playlists never emit events, preserving fixed-shader behaviour.
- `shuffle.rs` — `ShuffleBag` randomizer. Returns every index in `0..len` exactly once per bag cycle in a freshly randomized order; reshuffles on exhaustion; guarantees no cross-bag consecutive repeats when `len >= 2`. "iPod shuffle" pattern — uniform-over-cycle, not uniform-per-pick. A separate instance per cycle stream (shaders, palettes), each with its own xorshift64 seed. `seed_from_time()` helper for wall-clock seeding.
- `preview.rs` — Windowed preview mode with egui control panel. Left region: shader viewport. Right region: 300-px docked panel with Shader and Palette tabs and thumbnail previews. FPS counter is an overlay (top-left, toggled with `I`). Keyboard shortcuts: Space (pause/resume), ←/→ (prev/next shader), ↑/↓ (prev/next palette), R (reset time), F (toggle panel), I (toggle FPS), T (test shader crossfade), Q/Escape (quit).
- `render_preview.rs` — `render-preview` subcommand. Headless EGL surfaceless + FBO capture; encodes animated WebP. Defaults: 480×270, 3 s, 15 fps, quality 80. Batch mode (no shader names) renders all shaders. Per-shader palette resolution: CLI override → `[render_preview.palettes]` config map → stable hash-based default. `--skip-existing` skips outputs that already exist.
- `egl.rs` — Shared `EglState` (EGL instance/display/config) initialisation from a `wl_display` pointer, used by both `wayland.rs` and `preview.rs`. Distinct from `headless_egl.rs` — do not merge.
- `headless_egl.rs` — Surfaceless EGL context for `render-preview` (no Wayland surface needed).

Entry point: `main.rs` — CLI (clap), signal handling (signal-hook), config load, then dispatches to `preview.rs` (windowed preview) or `wayland.rs` (layer-shell screensaver). Event loop is calloop.

## Build Environment

This environment does not have all system libraries installed (notably `xkbcommon`). `cargo build` will fail at the linker stage — this is expected and not a code error.

After editing shader files (`.frag`, `.vert`) or Rust source:
1. Run `touch src/shaders.rs` to invalidate the cargo cache for shader changes
2. Do NOT run `cargo build` — it will fail on missing system deps
3. Do NOT attempt to install system packages
4. Commit changes and push to the current feature branch

## Build & Run
```sh
cargo build --release
./target/release/hyprsaver                          # screensaver mode (needs Hyprland)
./target/release/hyprsaver --preview oscilloscope   # windowed preview
./target/release/hyprsaver render-preview           # batch-render WebP previews of all shaders
./target/release/hyprsaver render-preview blob      # single-shader WebP preview
```

## Key Design Decisions
- **glow over wgpu**: Thin OpenGL wrapper, minimal complexity for v1. wgpu (Vulkan support) remains on the long-term roadmap.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maravexa/hyprsaver](https://github.com/maravexa/hyprsaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
