---
trigger: always_on
description: Bit-perfect music player (macOS + Linux). Pure Rust, Ratatui TUI. Four crates:
---

# Project Rules

## What is koan

Bit-perfect music player (macOS + Linux). Pure Rust, Ratatui TUI. Four crates:

- **koan-core** — library crate. Audio engine, player, database, indexer, format strings, file organization, remote (Subsonic/Navidrome) client, shared helpers. No UI code, no terminal deps.
- **koan-tui** — library crate. Ratatui TUI, visualizers, media keys, download queue. Exports `run_tui()`. Depends on koan-core.
- **koan-server** — library crate. GraphQL (async-graphql + axum), Subsonic REST API, MCP server. Depends on koan-core.
- **koan-cli** — binary crate (`koan`). Thin entry point: clap CLI, logger, signal handling, command routing. Depends on koan-core + koan-tui + koan-server.

Dependency rules (compiler-enforced): koan-tui and koan-server cannot import each other. Future iOS app imports only koan-core.

## Architecture overview

Read `ARCHITECTURE.md` for the full technical manual (threading model, data flow, sync primitives, module reference). This section is the quick-ref.

### Threading model (5 threads at steady state)

```
Main Thread (TUI, 60fps)   ──crossbeam channel──►  Player Thread ("koan-player")
                                                       │
                                                       ├──rtrb ring buffer──►  Decode Thread ("koan-decode")
                                                       │
                                                       └──controls──►  Audio RT Thread (CoreAudio/cpal, system-managed)

Analyzer Thread ("koan-analyzer") ◄──VizBuffer──  Decode Thread
                                  ──VizSnapshot──►  Main Thread (TUI)
```

**Golden rule: the audio render callback must NEVER allocate or lock.** It only touches atomics and the rtrb consumer.

### Sync primitives

| Data | Primitive | Why |
|------|-----------|-----|
| PCM samples (decode→audio output) | `rtrb` SPSC ring buffer | Lock-free, cache-friendly |
| Commands (TUI→Player) | `crossbeam-channel` bounded(16) | Backpressure, timeout recv |
| Atomics (position, state, samples_played) | `AtomicU8/U64/Bool` Relaxed | Hot path, no contention |
| Complex shared state (playlist, track info) | `parking_lot::RwLock` | Faster than std, no poisoning |
| Viz samples (decode→analyzer) | `VizBuffer` (`parking_lot::Mutex`) | Ring of f32 for FFT |
| Analysis output (analyzer→TUI) | `VizSnapshot` (`parking_lot::Mutex`) | Atomic snapshot |
| Parallel work (scan, remote sync) | `rayon` | Work-stealing thread pool |

### Key data flow

```
File → Symphonia → f32 → rtrb ring buffer → platform audio callback → DAC
```

No resampling. Device sample rate switched to match source (bit-perfect). Float32 all the way.

### Key design decisions

- **QueueItemId (UUIDv7)** — all queue ops use IDs, not indices. Survives reordering, handles duplicate tracks.
- **Status is derived** — `QueueEntryStatus` computed from cursor + load state, never stored.
- **Decode cursor ≠ UI cursor** — decode thread peeks ahead for gapless without moving the playlist cursor.
- **One `derive_visible_queue()` per frame** — cached snapshot, all render/mouse ops see consistent state.
- **Track dedup across sources** — local file + remote entry = one DB row. 3-strategy match: path → remote_id → content.
- **Figment-layered config** — defaults → `config.toml` → `config.local.toml` → `KOAN_*` env vars. Use `Config::update_base()` for base config writes, `patch_local(section, values)` for machine-specific updates to `config.local.toml`.

## Git

- **NEVER push tags.** Tags and releases are handled externally. Only push commits.
- Work in PRs, never push to main.
- Don't rebase on merge — we squash PRs.

## Build & check

```bash
just check    # cargo test + clippy -D warnings
just fmt      # cargo fmt
just cli      # cargo run --release -p koan-music -- <args>
just build    # cargo build --release
```

Pre-push hook (`.claude/settings.json`) runs `cargo fmt --all` + `cargo clippy --workspace -- -D warnings` before any `git push`. If clippy fails, fix before pushing.

**Zero warnings policy.** Fix all clippy/compiler/lint warnings immediately. Run fmt after every change.

## Where things live

### koan-core (`crates/koan-core/src/`)

| Module | What |
|--------|------|
| `audio/backend.rs` | `AudioBackend` + `AudioEngineHandle` traits — platform-agnostic audio output |
| `audio/coreaudio_backend.rs` | macOS `CoreAudioBackend` impl (wraps engine.rs + device.rs) |
| `audio/cpal_backend.rs` | Linux `CpalBackend` impl (ALSA/PipeWire/PulseAudio via cpal) |
| `audio/engine.rs` | CoreAudio AUHAL setup, render callback (macOS only) |
| `audio/buffer.rs` | `PlaybackTimeline`, track boundaries, decode thread entry points (`start_decode`, `decode_queue_loop`, `decode_single`) |
| `audio/device.rs` | CoreAudio device enumeration, sample rate get/set (macOS only) |
| `audio/replaygain.rs` | EBU R128 loudness scanning, gain application via lofty |
| `audio/viz.rs` | `VizBuffer` (ring of f32 samples for analyzer), `VizSnapshot` (atomic snapshot for UI) |
| `audio/analyzer.rs` | FFT analysis thread — 48-band spectrum, VU meters, peak hold. Runs at configurable FPS |
| `audio/streaming.rs` | Progressive download with `Condvar`-based ready signaling |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radiosilence/koan](https://github.com/radiosilence/koan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
