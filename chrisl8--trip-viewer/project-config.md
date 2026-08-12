---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development (hot-reload, ~10s incremental Rust builds)
npm run tauri dev

# Production build (creates NSIS installer at src-tauri/target/release/bundle/nsis/)
npm run tauri build

# Type-check TypeScript
npx tsc --noEmit

# Rust tests (35+ unit tests, mostly in src-tauri/src/import/ and src-tauri/src/scan/)
cargo test --manifest-path src-tauri/Cargo.toml

# Run a single Rust test (example)
cargo test --manifest-path src-tauri/Cargo.toml test_copy_and_hash_matches_hash_file

# Clippy (should be zero warnings — enforce this)
cargo clippy --manifest-path src-tauri/Cargo.toml -- -W clippy::all
```

## Architecture

Trip Viewer is a **Tauri v2** desktop app: Rust backend (`src-tauri/src/`) communicates with a React/TypeScript frontend (`src/`) via Tauri commands and events. Targets Windows (NSIS installer) and Linux (AppImage; Flatpak planned). Windows is the primary development platform; the Linux port relies on WebKitGTK 4.1 and GStreamer for video rendering.

### Rust backend module map

- **`scan/`** — folder scanner. Parses Wolf Box filenames (`YYYY_MM_DD_HHMMSS_EE_C.MP4`), fuzzy-matches triplets within a 3-second window, merges segments into trips with a 120s gap threshold. Uses `rayon` for parallel metadata probing.
- **`gps/`** — custom binary parser for the **ShenShu MetaData format** used by Wolf Box firmware. This was reverse-engineered — no upstream spec exists. Decodes NMEA DDMM.MMMM coordinates from the `gpmd` track.
- **`metadata/`** — MP4 probe using the `mp4` crate (pure Rust, **no ffprobe dependency** — this is a locked decision, see DESIGN.md).
- **`import/`** — SD card import pipeline (10 submodules). **Safety-critical**: files are SHA-256 hashed during copy, re-hashed on the destination, and the source is only wiped after every file is verified. See "Import pipeline invariants" below.
- **`timelapse/`** — background ffmpeg-driven pre-render pipeline that produces 8x / 16x / 60x fast-playback MP4s per (trip, tier, channel). ffmpeg is an opt-in user dependency (configured via the `settings` table, not bundled). NVENC + NVDEC path is used when available; see `ffmpeg.rs::Encoder::needs_cuda_hwaccel`. **Event-detection thresholds for the variable-speed 16x/60x tiers live in `events.rs` — see that module's top-of-file doc comment for how to verify the slowdown behavior after a run and how to tune thresholds.**
- **`error.rs`** — `AppError` enum with `thiserror`; implements `Serialize` for automatic JSON conversion to the frontend.

### Frontend structure

- **`App.tsx`** — sidebar (trip list, SD + folder import buttons, storage summary, version footer) plus a top tab bar (`MainNavTabs`) that switches between Player, Scan, Review, Timelapse, optional Issues, and Places. The Issues tab only renders when there are scan errors; Places lives at the right of the bar.
- **`components/video/`** — `VideoGrid`, `ChannelPanel`, `PlayerShell`. The grid renders one `<video>` per channel reported by the active segment (1 to 4). Channels are **always rendered in stable DOM order** for the dashcam's kind set; swap behavior uses CSS grid placement only. Moving them in the tree would cause React to unmount/remount the `<video>` elements and pause playback.
- **`components/import/`** — confirm dialog, progress UI, unknown files dialog, summary. Progress events stream from Rust via `window.emit()`, frontend listens with `@tauri-apps/api/event`. There are two import flows: SD-card (destructive, wipes source) and folder (non-destructive).
- **`components/timelapse/`** — Timelapse Library view, ffmpeg config modal, per-trip rebuild, scope picker (new & unfinished / retry failed / rebuild all).
- **`engine/useSyncEngine.ts`** — video sync engine. Uses `requestVideoFrameCallback` against a stable master ref (front when present, otherwise the first channel); other channels are slaved to it. Master ref is the timing master regardless of which channel is visually primary.
- **`state/store.ts`** — Zustand store with `LibrarySlice`, `PlaybackSlice`, `ImportSlice`, plus timelapse state. `primaryChannel` controls layout but not sync.

## Locked architectural decisions (do not revisit without strong reason)

See DESIGN.md for full context. Key ones:

- **HTML5 `<video>` for playback** (not libmpv). `tauri-plugin-libmpv` is broken for multi-instance on Windows.
- **Pure Rust `mp4` crate** (not ffprobe). Bundling ffprobe adds 80 MB and triggers Defender heuristics.
- **HEVC Extension tax accepted** — app uses a `<HevcSupportGate>` startup check with Store deep-link on Windows, and an apt-install hint on Linux when GStreamer's libav plugin is missing.
- **NSIS on Windows, AppImage on Linux (Flatpak planned).** MSI rejected (~130 MB vs 3 MB NSIS). `.deb` skipped — AppImage bundles its own GStreamer plugins for codec-complete direct downloads. A future Flatpak would reach Debian/Ubuntu/Fedora/Arch with bundled codecs via `org.freedesktop.Platform.ffmpeg-full`, but no Flathub manifest exists yet.
- **No fullscreen API on single-click** — use double-click (conflict with play/pause expectation).

## Import pipeline invariants


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisl8/trip-viewer](https://github.com/chrisl8/trip-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
