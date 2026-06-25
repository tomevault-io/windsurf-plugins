---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

MeterMaid is a cross-platform desktop **LUFS / loudness meter** built with **Tauri 2** — a Rust audio engine (`src-tauri/`) plus a vanilla TypeScript + Vite web UI (`src/`). No frontend framework; the UI is a single `index.html` + `src/main.ts` driving a canvas spectrum. Developed primarily on macOS.

## Commands

```sh
pnpm install              # also installs the git pre-commit hook via core.hooksPath
pnpm tauri dev            # run the app (Vite dev server + Rust); HMR applies UI edits live
pnpm tauri build          # production bundle → src-tauri/target/release/bundle/
pnpm build                # frontend type-check + Vite build only (tsc && vite build)
pnpm lint                 # Biome (TS) + markdownlint (Markdown)
pnpm format               # apply Biome fixes/formatting
```

Rust checks run from `src-tauri/`:

```sh
cargo clippy --all-targets --all-features -- -D warnings
cargo test
cargo test build_error_messages_name_the_device   # single test by name substring
cargo test ebur128_matches_ffmpeg -- --ignored --nocapture   # optional ffmpeg cross-check (needs ffmpeg)
```

The audio tests in `audio.rs` (`#[cfg(test)] mod tests`) drive the `Analyzer` directly with synthesized frames — **no audio device required**. New analysis behavior should come with one of these golden-signal tests.

CI (`.github/workflows/ci.yml`) runs `pnpm build`, `cargo fmt --check`, clippy, and `cargo test` on macOS + Linux, plus a `cargo audit`. The pre-commit hook lints only *staged* `.rs`/`.ts`/`.md`; bypass with `git commit --no-verify`, skip just clippy with `SKIP_CLIPPY=1 git commit`.

## Architecture

**The realtime-safety contract is the central design constraint.** `src-tauri/src/audio.rs` is built around keeping the audio callback allocation- and lock-free:

- A single dedicated thread, `engine_loop`, owns the `cpal::Stream` (cpal streams are `!Send`) and is the **sole owner** of the `Analyzer` — so the analyzer needs no synchronization.
- The realtime cpal callback does no locking/allocation in steady state: it only pushes incoming frames into a lock-free SPSC ring (`ringbuf::HeapRb`) and tallies dropped samples with a relaxed atomic on overrun. **Never add allocation, locking, or logging to this callback.**
- `engine_loop` blocks on `rx.recv_timeout(EMIT_INTERVAL)` (33 ms). On each timeout it drains the ring, de-interleaves the user-selected channel(s), feeds an `ebur128` analyzer (BS.1770 loudness) and a mono ring for the `rustfft` spectrum, then emits a `meter-update` event with a `Metrics` struct. A separate emit thread (bounded `sync_channel(1)`, coalescing) does the Tauri `emit` so a slow UI can never stall the drain.

**Frontend ↔ backend IPC** has two directions:

- **Commands** (`src/main.ts` `invoke(...)` → `lib.rs` `#[tauri::command]`): `list_devices`, `get_device_config`, `start_capture`, `stop_capture`, `reset_integrated`. The capture commands forward a `Command` enum over an `mpsc::Sender` to `engine_loop` and wait on a reply `SyncSender` — i.e. command handlers are thin shims; the engine thread does the work.
- **Events** (`lib.rs`/`audio.rs` `emit` → `src/main.ts` `listen`): `meter-update` (per-frame metrics) and `stream-error` (OS stream fault, e.g. device unplugged mid-capture → UI tears down and surfaces the reason).

Rust `Metrics`/`DeviceConfig`/`StreamInfo` use `#[serde(rename_all = "camelCase")]`; the matching TS `interface`s in `main.ts` must stay in sync (snake_case Rust field ↔ camelCase TS field).

**Frontend rendering** is a `requestAnimationFrame` loop (`frame`/`requestFrame` in `main.ts`) that draws the spectrum canvas. It is **gated on capture state**: it self-sustains only while `running`, and otherwise repaints exactly once per state change (start, stop/teardown, resize, load). Keep that gating intact — an always-on rAF loop redraws a static plot at the display refresh rate and burns several percent of idle CPU/GPU. Any new state that changes the canvas while idle must call `requestFrame()`.

**Device handling.** `list_input_devices` enumerates via cpal, which exposes only a name (no transport/type) — so the picker deliberately shows *all* inputs and cannot reliably filter microphones. cpal has no hotplug callback, so `main.ts` polls `list_devices` every 2 s while idle (`pollDevices`), rebuilding the dropdown only when the device set changes and preserving the selection. `device_config` only offers candidate sample rates from supported-config ranges that match the **default config's channel count and sample format** (what `build_stream` actually uses); offering others would let Start fail with `StreamConfigNotSupported`.

**Settings persistence** uses `@tauri-apps/plugin-store` (`settings.json`): device, channels, sample rate, target LUFS, clip ceiling — all re-validated against present hardware on load (missing device → system default with a notice). Window geometry persists via `tauri-plugin-window-state`; a small `window_guard_plugin` in `lib.rs` recenters the window if its last monitor is gone.

## macOS entitlement gotcha


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reverentgeek/metermaid](https://github.com/reverentgeek/metermaid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
