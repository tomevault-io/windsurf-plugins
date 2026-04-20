---
trigger: always_on
description: `waydeeper` is a GPU-accelerated depth effect wallpaper daemon for Wayland compositors. Uses ML-based monocular depth estimation (ONNX) to create a parallax effect where wallpaper layers shift as the mouse moves. An optional `--inpaint` mode uses 3D-photo-inpainting to generate a 3D mesh with correct occlusion.
---

# AGENTS.md - waydeeper

## What This Is

`waydeeper` is a GPU-accelerated depth effect wallpaper daemon for Wayland compositors. Uses ML-based monocular depth estimation (ONNX) to create a parallax effect where wallpaper layers shift as the mouse moves. An optional `--inpaint` mode uses 3D-photo-inpainting to generate a 3D mesh with correct occlusion.

## Current Status — Fully Working

All CLI commands and the full rendering pipeline are functional. Tested on niri compositor with both integer and fractional HiDPI scaling.

**Working:**
- Full rendering pipeline: Wayland layer-shell + EGL + OpenGL ES 3.0
- GPU-accelerated parallax depth effect (GLSL ES 300 shaders)
- 3D inpainting mode: two-pass rendering (flat background + 3D mesh) with UV-texture sampling from full-res wallpaper
- Fractional HiDPI scaling via `wp_fractional_scale_v1` + `wp_viewporter`
- Multi-monitor support (independent daemon subprocess per output)
- All CLI commands: `set`, `daemon`, `stop`, `list-monitors`, `pregenerate`, `cache --list/--clear`, `download-model`, `download-model inpaint`
- ONNX depth estimation via `ort` crate (load-dynamic, system libonnxruntime)
- Depth map caching with blake2b hashing, model-aware cache keys
- Inpaint PLY mesh caching with blake2b hashing (image + depth + config)
- Unix domain socket IPC (PING/STATUS/STOP/RELOAD)
- Subprocess-based daemon spawning (parent waits for wallpaper ready, then exits)
- Background reload: daemon regenerates assets in a background thread while the renderer continues, then swaps textures/mesh in-place
- Signal handling (SIGTERM via `nix::sys::signal::sigaction`)
- Smooth animation with configurable delay/idle timers
- Proxy support (HTTP_PROXY/HTTPS_PROXY/ALL_PROXY/NO_PROXY) for all downloads
- Nix flake build (`nix build`, `nix develop`)
- Zero compiler warnings

## Architecture

```
src/
  main.rs            - Entry point, dispatches to cli::run()
  cli.rs             - Clap CLI. spawn_daemon() forks subprocess with "daemon-run" command
                       Proxy-aware download helper (HTTP_PROXY/HTTPS_PROXY/NO_PROXY)
                       wait_for_daemon() with spinner animation, 180s timeout
                       send_reload() polls STATUS for progress during background reload
  config.rs          - JSON config (~/.config/waydeeper/config.json)
  models.rs          - Model registry (depth-anything-v3-base, midas-small, depth-pro-q4)
                       inpaint_models_dir(), inpaint_models_present()
  cache.rs           - DepthCache with blake2b hashing, 16-bit PNG depth map I/O
                       InpaintCache for PLY mesh caching
  ipc.rs             - DaemonSocket (server) / DaemonClient (client) over Unix sockets
                       ReloadState for tracking background reload progress
  depth_estimator.rs - ort crate ONNX wrapper, Lanczos3 resize, Gaussian blur
  daemon.rs          - DepthWallpaperDaemon: depth/inpaint → IPC → renderer (in order)
                       run_daemon_loop() handles reload state machine
  inpaint.rs         - Python subprocess launcher (stdout/stderr streaming)
  mesh.rs            - Binary/ASCII PLY parser with UV coords, image_aspect, fov_y_deg
  math.rs            - perspective() and translation() 4×4 column-major matrix helpers
  renderer.rs        - Dual-mode renderer (flat depth-warp + mesh perspective)
                       Two-pass draw: flat background quad + mesh with back-face culling
                       Mesh shader samples full-res wallpaper via UV coords (not baked vertex colors)
                       Fragment shader culls stretched triangles (depth gradient > 0.08)
                       reload_textures() and reload_mesh() for in-place swap during reload
  wayland.rs         - smithay-client-toolkit: layer-shell, pointer tracking, fractional scale
                       OutputProbe for list_connected_outputs() (monitor availability check)
                       Reload asset generation in background thread, in-place texture swap
  egl_bridge.c       - ~100 lines C: EGL init from wl_display, window surface via wl_egl_window
build.rs             - Compiles egl_bridge.c, links libEGL + libwayland-egl
scripts/
  inpaint.py         - Full 3D inpainting pipeline:
                       Bilateral smoothing, edge/depth/color ML inpainting, graph-based mesh builder
                       Graph-based topology with edge tearing at depth discontinuities
                       Dangling edge removal and component filtering (≥200 nodes)
                       Binary PLY output with per-vertex UV texture coordinates
  networks.py        - Neural network architectures (MIT, from 3d-photo-inpainting)
```

## Key Design Decisions

1. **Subprocess spawning**: `cmd_set`/`cmd_daemon` spawn the binary as a subprocess with the hidden `daemon-run` subcommand. Parent waits for IPC responsiveness (max 180s) then exits. Each monitor gets its own subprocess. Daemon inherits stdout/stderr for progress reporting.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EdenQwQ/waydeeper](https://github.com/EdenQwQ/waydeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
