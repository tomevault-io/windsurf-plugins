---
trigger: always_on
description: driftwm — a trackpad-first infinite canvas Wayland compositor written in Rust. Windows float on an unbounded 2D plane navigated via trackpad gestures (pan, zoom, pinch). No workspaces, no tiling. Built on [smithay](https://github.com/Smithay/smithay).
---

# CLAUDE.md

## Project

driftwm — a trackpad-first infinite canvas Wayland compositor written in Rust. Windows float on an unbounded 2D plane navigated via trackpad gestures (pan, zoom, pinch). No workspaces, no tiling. Built on [smithay](https://github.com/Smithay/smithay).

The project is launched (v0.1.x). See `docs/DESIGN.md` for the full specification and `docs/CAVEATS.md` for architectural pitfalls.

## Conventions

- Documentation files (except README.md) live in `docs/`.
- Config path: `~/.config/driftwm/config.toml` (respects `XDG_CONFIG_HOME`).

## Code Style

- Write self-documenting code: clear names, obvious structure, minimal comments.
- No section-separator comments (e.g. `// ---- Protocols ----` or `// === Input ===`). Code structure should be clear from the code itself.
- Comments explain *why*, not *what*. Don't restate what the code does.
- Brief doc comments (`///`) on public functions are fine when the signature isn't self-explanatory.
- Inline comments for non-obvious logic (smithay quirks, coordinate space tricks) are good.

## Build & Run

```bash
cargo build              # build
cargo run                # run nested in existing Wayland session (winit backend)
cargo run -- --backend udev   # run on real hardware (from TTY)
cargo test               # run tests
cargo test test_name     # run a single test
cargo clippy             # lint
```

Use `RUST_LOG=debug cargo run` for smithay/libinput event traces.

Udev backend build deps (Fedora): `libseat-devel libdisplay-info-devel libinput-devel mesa-libgbm-devel`.

### Cross-distro build testing (containers)

Use podman to test builds on other distros (Docker Desktop is flaky on Fedora):

```bash
# Arch Linux
podman run --rm -it --security-opt label=disable -v ~/Documents/work/scripts/driftwm:/src archlinux:latest bash
pacman -Syu --noconfirm rust cargo pkg-config libdisplay-info libinput seatd mesa libxkbcommon
cd /src && cargo build
```

Notes:
- `--security-opt label=disable` is required on Fedora (SELinux blocks libc inside container otherwise).
- Use `cargo build` (not `--release`) for faster dep checking — release optimizations are slow and unnecessary for verifying deps.
- Don't copy the repo inside the container — `target/` is huge. Mount directly without `:ro` so cargo can write to `target/`.

## Architecture

The compositor uses a **camera/viewport** model: the screen is a viewport onto an infinite 2D plane. Each window has absolute `(x, y)` canvas coordinates. The viewport has a camera `(cx, cy)` and zoom `z`. Screen position = `(wx - cx) * z`. Multiple monitors = multiple independent viewports on the same canvas.

Current source layout:

- `main.rs` — entry point (CLI args, backend selection), `lib.rs` — crate root (module declarations)
- `backend/` — `mod.rs` (Backend enum: Winit/Udev + renderer accessor), `winit.rs` (winit backend init + ~60fps timer render loop), `udev.rs` (udev/DRM backend init + VBlank-driven render loop, libseat session, libinput, hotplug)
- `state/` — `mod.rs` (DriftWm struct, FullscreenState, ClientState), `animation.rs` (camera/zoom/momentum/edge-pan animation, key repeat), `navigation.rs` (navigate_to_window, focus history, MRU cycle), `fullscreen.rs` (enter/exit fullscreen, pointer remap), `fit.rs` (per-window fit-to-viewport toggle, pre-fit size restore)
- `config/` — `mod.rs` (Config struct, load/parse, context-aware lookup methods), `types.rs` (Action, Direction, Modifiers, KeyCombo, MouseBinding/MouseTrigger/MouseAction, GestureBinding/GestureTrigger, ContinuousAction/ThresholdAction, ContextBindings, BindingContext), `parse.rs` (string→type parsers for combos/actions/gestures), `defaults.rs` (default key/mouse/gesture bindings per context, terminal/launcher detection), `toml.rs` (serde structs, config path)
- `canvas.rs` — coordinate transforms (ScreenPos/CanvasPos), camera math, cone search, zoom helpers (zoom_to_fit, zoom_anchor_camera, snap_zoom, dynamic_min_zoom)
- `focus.rs` — FocusTarget(WlSurface) newtype with KeyboardTarget/PointerTarget/TouchTarget impls
- `decorations.rs` — per-window SSD state, CPU-rendered title bar, hit-testing helpers
- `render.rs` — OutputRenderElements, compose_frame(), post_render(), update_background_element(), tile/cursor/layer rendering helpers
- `shaders/` — GLSL shader source files (dot_grid, shadow, blur_down/blur_up/blur_mask, corner_clip, tile_bg)
- `snap.rs` — window snapping (magnetic edge alignment during drag)
- `window_ext.rs` — `WindowExt` trait for Wayland/X11 window polymorphism
- `input/` — `mod.rs` (keyboard handling, pointer motion absolute+relative, surface_under hit-testing), `actions.rs` (execute_action dispatch for all keybindings), `pointer.rs` (context-aware mouse dispatch, button/axis handling, compositor resize/pan grabs), `gestures.rs` (table-driven gesture dispatch from config, continuous/threshold state machine, libinput device config, client forwarding)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [malbiruk/driftwm](https://github.com/malbiruk/driftwm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
