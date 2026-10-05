---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Quest Pro Touch Plus (qptp): a Windows desktop app (Tauri 2 + Vue 3) that shows the extra touch-pad sensors of Quest Pro Touch Pro controllers (X, Y, force) and per-controller status. It receives a stream from [qptp-module](https://github.com/Lateir/qptp-module), which runs on the headset. The "Режим" view selects touchpad, one-button or two-button behavior for both controllers. The OpenVR auxiliary driver in `openvr-aux/` receives live values from the app over loopback UDP and exposes a single Treadmill-role input device alongside the Quest Pro controllers, plus two input-less OptOut devices that only show each controller's battery. Input must stay on the Treadmill device: OptOut devices get no `/user/...` path, so SteamVR never routes their input to bindings.

The README is in English. User-facing UI strings live in `src/i18n.ts` for the 12 supported languages; add every new string to every locale. The first launch chooses the first supported Windows preferred UI language, falling back to English; later launches use the saved choice. Browser preview uses `navigator.languages`. Arabic uses right-to-left text while left/right controllers keep their physical positions. Native tray menu labels are translated in Rust.

## Commands

```powershell
npm install
npm run tauri dev                       # full app (starts Vite on :5173 via beforeDevCommand)
npm run build                           # frontend only: vue-tsc -b type-check + vite build -> dist/
npm run tauri build                     # installer in src-tauri/target/release/bundle
npm run tauri build -- --no-bundle      # release exe without installer
cargo check --manifest-path src-tauri/Cargo.toml   # Rust-only check
```

The project has Rust unit tests and no linter. `vue-tsc -b` (part of `npm run build`) checks the frontend; `cargo test --lib` checks Rust.

## Architecture

Almost all logic is in two files. Both use a dense, compact style with many statements per line. Match it when editing.

**Backend: `src-tauri/src/lib.rs`**
- A single shared `StreamState` holds a `Mutex<Snapshot>`, a settings lock and an `AtomicU64 generation`. Tauri commands: `get_stream_state`, `set_transport(transport)`, `set_input_mode(mode)`, `set_press_threshold(threshold)`, `set_haptic_amplitude(amplitude)`, `set_language(language)`, `open_module_page`.
- The connection is always on: `setup` starts a `worker` thread with the saved transport. Transport, input mode, press threshold, haptic amplitude and language are saved to `<app_config_dir>/config.json`; the old `transport` file is read as a migration fallback. `set_transport` restarts the worker. Starting a worker bumps `generation`. Every loop and read (`read_exact_checked`, `read_frame`, `worker`) checks whether its captured generation is still current and exits when it is not. This is the cancellation mechanism. Preserve these checks when adding blocking steps.
- All state changes go through `publish()`, which mutates the snapshot and emits the `stream-state` event with the full snapshot. `QPR3` sensor updates are throttled to about one emit every 33 ms.
- `phase` is one of `searching | connecting | connected`. The frontend also sets `error` locally when an invoke fails. The worker retries forever, about once per second.
- Button modes are evaluated on every QPR2 frame in Rust. Default press threshold is 0.30; release threshold is fixed at 0.20 or loss of touch. In two-button mode Y < 128 selects the top half, Y >= 128 the bottom, and that selection latches until release. Button state is included in `Snapshot` and changes bypass the regular 33 ms UI throttle. New presses send a QPC1 thumb haptic pulse. Each controller has at most one outstanding pulse, cleared by its QPA1 reply; releases do not vibrate. `extra_ipc.rs` publishes the selected mode to the qptp driver via loopback UDP and records its acknowledgement for the SteamVR status display.
- Transports:
  - **USB**: runs `adb forward --no-rebind tcp:27182 tcp:27182`. If that fails, it accepts an existing matching forward. It removes the forward on stop only if it created it. Only the bundled adb is used: `<resource_dir>/resources/platform-tools/adb.exe`, shipped via `bundle.resources` in `tauri.conf.json`. It is spawned with `CREATE_NO_WINDOW`. On exit the app runs `adb kill-server` if adb was used.
  - **LAN**: discovery only, with no manual IP. It broadcasts UDP `QPD1` to port 27183 for up to 3 s. The reply is 8 bytes: `QPO1` + u16 LE port + `[1,0]`.
- Stream protocol (TCP, little-endian), ported from `qpro_debug/qpro_stream_receiver.py`:
  - `QPS1` + u32 size (≤4096) + JSON status body.
  - `QPV1` + u32 JSON length + JSON `version`/`versionCode` on connect. Module `versionCode` below 9 requires an update.
  - `QPR3` + 52-byte body. Left sensor at bytes 12–31, right at 32–51. Each side has u16 x/y and f32 force, stylus, trigger proximity, and trigger slide. `QPR2` marks the module as too old.
  - Any other header is an error and triggers a reconnect.
  - Y is inverted in `sensor()` (`255 - raw`), so 0 is the top of the pad. The frontend draws it as is.
- Tray and window lifecycle:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lateir/qptp](https://github.com/Lateir/qptp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
