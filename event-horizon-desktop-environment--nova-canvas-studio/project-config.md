---
trigger: always_on
description: C++20 / Qt6 / FFmpeg **nonlinear video editor** (dark, editor-grade UI). Linux-only. The GUI shell lives in `gui/`, the headless editor/export engine in `core/` (`canvas_core` static lib). Namespaces: `canvas::core` / `canvas::gui` (GPU code is `canvas::core::gpu`).
---

# AGENTS.md — Nova Canvas Studio

C++20 / Qt6 / FFmpeg **nonlinear video editor** (dark, editor-grade UI). Linux-only. The GUI shell lives in `gui/`, the headless editor/export engine in `core/` (`canvas_core` static lib). Namespaces: `canvas::core` / `canvas::gui` (GPU code is `canvas::core::gpu`).

Renamed from **Event Horizon Studio** to **Nova Canvas Studio** on 2026-09-05 (binary `canvas`). Project files write the `canvas_project` key; older files with the legacy `event_horizon_project` key still load.

Mid-refactor: `splitplan.md` is a 40-phase plan to split the codebase into small, Qt-free-where-possible, independently-testable modules. **Phases 1–30 are done** (headless extraction of the playback stack + timeline interaction, see below); phases 31–40 (Deliver-panel split, audio-path dedup, dead-code removal, `-Werror` hardening, this file) are still open. Check `splitplan.md`'s checkbox state before assuming a module described below hasn't moved again.

## Build, run, test

- **Build:** `./build.sh` (also `-d` install deps, `-c` clean, `-t Debug`). Binary → `build/gui/canvas`. Auto-detects distro, prefers Ninja.
- **Clean code only, zero warnings:** every build — Debug, Release, `build-release/` — must be warning-free. Treat any compiler warning as a build failure before handing off. NOTE: `-Werror` is **not enabled on any target yet** (splitplan Phase 38 is open; audited 2026-09-07), so zero-warning discipline is manual everywhere — treat every warning as a failure.
- **Test:** `ctest --test-dir build`. **35 tests** (verified passing on this machine except `equalizer`, a user-WIP failure in the Inspector EQ — see note below):
  - **Core (18, in `core/tests/`):** `roundtrip` (edit-op + project JSON round-trip; an edge in this test used to SEGFAULT on a stale timeline model until `EqBand::operator==` was added — it now passes), `export_sweep` (every valid codec×container combo; skip=2 if no libx264), `scrub_bench` (p95 scrub-preview latency budget; skip=2 if no libx264), `colorsci`, `graph`, `graph_edit`, `composite`, `op`, `lut` (Resolve-style 3D grade-LUT bake: identity byte-exact round-trip, gain byte-exact vs the evaluator, grid-vs-evaluator parity at gridpoints, empty/unwired → null), `gpu_grade` (pins the fused CUDA grade kernel's law — `nv12GradeResize` — by host-side mirror, incl. identity/limited-range/CPU-export parity, no device needed), `wheels_ui`, `curves`, `histogram`, `visual_render_test` (eye: identity video render is byte-identical to the legacy fast path, plus flip/scale/opacity actually change output pixels; synthesizes its own h264 source at runtime), `voice_isolation_test` (headless RNNoise seam: engine + per-clip field + linked-mate edit op + JSON round-trip on synthetic PCM), `clip_rate_test` (per-clip speed law/pitch constants + the pitch-preserving WSOLA engine: 2× keeps 440 Hz pitch, 0.5× length, chunked≡whole, stereo, bank hygiene), `equalizer` (EqBank/EqBand laws — currently red on the "mid-stream gain drag stays sample-continuous" check while the Inspector-EQ WIP is mid-flight), `vram_leak` (VRAM regression gate: reproduces NVDEC decode of a synthetic 2K h264 at native res + host download sampling `cudaMemGetInfo`; fail if steady-state free VRAM drains >256 MB — pinned the `av_frame_ref`-onto-dirty-`retain_hw_` leak that filled all 16 GB in ~90 s of playback).
  - **GUI-headless (14, in `gui/tests/`):** `sync_constants_test`, `timeline_decoder_test`, `transition_bake_test` (off-thread transition pre-render discovery — one-frame dissolve on same-media lead; the "unbreakable seam" Qt-free unit), `audio_pipeline_test`, `sonicsync_test`, `av_reanchor_test` (seek-hold vs. audio-feed integration), `timeline_snap_test`, `timeline_selection_test`, `timeline_drag_test`, `transition_handle_editor_test`, `audio_targets_test` (multi-select audio-target resolution), `timeline_volume_line_test` (audio volume-line dB↔y law), `deliver_settings_model_test` (Deliver-panel container↔codec policy + bitrate-visibility law), `source_preview_model_test` (Dual-Viewer media-pool-scrub source-preview model: pool-backed single-clip project + scrub fraction↔source-frame law).
  - **GUI Qt-linked (3, in `gui/tests/qt/`):** `volume_line_drag_qt_test` — drives the real `TimelineWidget` press/move/release handlers in an offscreen session; `waveform_placement_qt_test`; `wheel_panel_roundtrip_qt_test` (the only Qt-linked tests; `scripts/check_qtdep.sh` exempts `gui/tests/qt/` by design).
  - If you quote a test count elsewhere, assume these 35 and re-verify with `ctest --test-dir build` before quoting.
- **`scripts/check_qtdep.sh`** — static guard for the headless invariant below (`-q` for exit-code-only, used by CI/`build.sh`). Also enforced at compile time: `canvas_add_headless_test()` targets link no Qt, so a stray `<Q...>` include fails the build outright.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Event-Horizon-desktop-environment/Nova-Canvas-Studio](https://github.com/Event-Horizon-desktop-environment/Nova-Canvas-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
