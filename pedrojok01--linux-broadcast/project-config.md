---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

Background-replacement virtual webcam for Linux. Captures a webcam frame, runs MediaPipe / RVM segmentation through `ort` (ONNX Runtime) — on CPU by default, or on an NVIDIA GPU via the optional `cuda` feature — composites the foreground over a blurred background, a saved image, or passes the frame through unchanged, and writes the result to a `v4l2loopback` virtual camera that Zoom / Meet / Teams / Firefox / OBS consume.

Out of scope: audio / microphone effects.

## Common commands

```bash
# Manual host setup — only needed when building from source. The .deb's
# postinst runs the same commands and ships conffiles in /etc/modprobe.d/
# + /etc/modules-load.d/ so the module persists.
sudo modprobe -r v4l2loopback 2>/dev/null
sudo modprobe v4l2loopback devices=1 video_nr=10 card_label="LinuxBroadcast" \
  exclusive_caps=1 max_buffers=2

# GUI (default)
cargo run --release -p linux-broadcast

# Headless — starts hidden in tray and auto-starts the pipeline. Used by
# the autostart .desktop. Both forms work.
cargo run --release -p linux-broadcast -- --headless
LB_HEADLESS=1 cargo run --release -p linux-broadcast

# Dump the bundled window icon to /tmp/lb-icon.png. Same code path that
# regenerates packaging/LinuxBroadcast.png when the logo changes.
LB_DUMP_ICON=1 cargo run --release -p linux-broadcast

# Lint / format
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all

# Build a local .deb (target/debian/linux-broadcast_<ver>-1_amd64.deb).
cargo install cargo-deb   # one-time
cargo deb -p linux-broadcast

# GPU build / packages. The cuda feature is off by default; ORT_CUDA_VERSION=13
# is supplied by .cargo/config.toml. See "GPU acceleration" below.
cargo run --release -p linux-broadcast --features cuda
packaging/build-cuda-addon.sh   # builds the linux-broadcast-cuda add-on .deb

# Cut a release. Precondition: your changes are already committed AND
# pushed to origin/main — the script refuses a dirty tree or an
# unpushed-ahead main (it only adds the version bump + tag on top).
# Bumps Cargo.toml + path-dep + Cargo.lock, runs the CI gates, commits
# "Release X.Y.Z", tags vX.Y.Z. --push sends the commit + tag; the tag
# triggers release.yml, which validates tag==version (else fails), then
# builds the .deb + GPU add-on, publishes the GitHub Release, and pushes
# to AUR. So the bump is mandatory — a bare tag would fail CI.
scripts/release.sh 0.1.3 --push

# Verify the virtual cam from another terminal (only works while the
# pipeline is running — exclusive_caps=1 hides /dev/video10 otherwise).
ffplay -fflags nobuffer -f v4l2 -input_format yuyv422 \
  -video_size 1280x720 /dev/video10

# Profiling / throughput. LB_PROFILE=1 prints per-stage feeder timings
# (pull / segment / composite / push + effective FPS) to stderr while
# Live. LB_INFER_INTERVAL=N runs segmentation once every N Live frames
# and reuses the cached mask on the rest (default 1 = every frame; 2 ≈
# half the inference cost for a ~1-frame mask lag during motion).
# Headless benches under crates/pipeline/examples/: bench.rs (full feeder
# loop, needs an ffmpeg consumer), infer_bench.rs (segment-only, CPU vs GPU),
# composite_bench.rs (composite stage per background mode).
LB_PROFILE=1 LB_INFER_INTERVAL=2 cargo run --release -p linux-broadcast

# Source codec. The feeder auto-selects MJPEG (3× framerate on USB cams:
# C920 720p is 30 fps MJPEG vs 10 fps raw YUYV) and falls back to raw if
# the camera can't negotiate it. LB_FORCE_RAW=1 skips the MJPEG attempt
# (escape hatch for cameras with a bad hardware JPEG encoder).
LB_FORCE_RAW=1 cargo run --release -p linux-broadcast
```

System dev packages are listed in [README §Build from source](README.md#option-b--build-from-source). Pin `v4l2loopback-dkms ≥ 0.12.8` — 0.12.7 fails to build on kernel 6.8+.

## Architecture

Cargo workspace, two crates:

- **`crates/pipeline`** (lib `lb_pipeline`) — the entire video pipeline, headless, no GUI deps.
- **`crates/app`** (bin `linux-broadcast`) — `eframe`/`egui` GUI driving the pipeline. Owns config persistence, the saved-background library, the theme, autostart, tray.

### Frame pipeline (two GStreamer graphs, one feeder, lazy by default)

`/dev/video0` is only opened while a real consumer is reading `/dev/video10` or the GUI preview pane is visible. To do that without `/dev/video10` blinking out of conferencing-app device lists, the pipeline is split into two GStreamer graphs glued by a Rust feeder thread.

```
                          ┌──────────────────────┐
                          │ consumer_watch       │  /proc/*/fd poll @ ~1.25 Hz
                          │ thread               │  → Vec<Consumer> on changes
                          └──────────┬───────────┘
                                     │
                                     ▼
   ┌────────────  feeder thread (lazy::Feeder) ─────────────────────────┐
   │  state machine: Idle → Activating(2s debounce) → Live              │
   │                       Live → Deactivating(3s debounce) → Idle      │
   │  demand = consumers ∪ gui_preview_active                           │
   │  owns: Segmenter, Compositor, MaskSmoother, Background slot        │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pedrojok01/linux-broadcast](https://github.com/Pedrojok01/linux-broadcast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
