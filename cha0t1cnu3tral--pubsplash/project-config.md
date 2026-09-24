---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Pubsplash is an accessibility-first Windows streaming client for Audio Pub (an Icecast-based livestreaming platform, server source: github.com/the-byte-bender/audiopub-sv), built with wxDragon (wxWidgets bindings). Its users are primarily screen-reader users; every control must be Tab-reachable and properly announced. The full product spec is in `project2.md` (`project.md` is an earlier draft of the same spec).

## Commands

- Build: `cargo build` (first build compiles wxWidgets via CMake/Ninja and takes several minutes; later builds are fast)
- Test: `cargo test`
- Single test: `cargo test <name_substring>`
- Ignored tests (hit the real audiopub.site or play audio): `cargo test <name> -- --include-ignored`. Notable: `real_login_bad_credentials` (live server), `sapi_synthesizes_pcm`
- Run: `./target/debug/pubsplash.exe` (release builds hide the console via `windows_subsystem`)

Per `agents.md`: every change set must update `changelog.md` (bulleted entries under `## Unreleased` → `### Additions`/`### Fixes`/`### Changes`) and keep `README.md` accurate (especially the shortcut table).

## Architecture

Three long-lived domains connected by channels; the UI never blocks on audio or network:

1. **UI (main thread)** — `src/ui/`. wxdragon main loop. A 100 ms `Timer` ("the pump", in `ui/mod.rs::pump_events`) drains `NetEvent`/`EngineEvent` receivers into UI state; once a second it refreshes durations and chat relative times. `App` (an `Rc` holding `RefCell`s) is the shared UI-side state: persisted `Config`, transient `Runtime`, handles to the engine/net/speaker threads, and `Widgets` (populated after `ui::build`).
2. **Audio engine (thread)** — `src/audio/`. `engine_loop` mixes 10 ms blocks: per-source WASAPI capture threads push f32 into rtrb rings → per-source `ChannelStrip` (volume + 50 ms mute fades) → master strip → optional LAME MP3 encode → tokio mpsc to the Icecast sender. Sources that aren't OS-captured (TTS, sound events) are `FeedKind::External`: the engine parks an rtrb producer in `ExternalFeeds` keyed by source name, and other subsystems push samples via `ExternalFeeds::push` (retry on `Full`, abort on `Gone`).

`src/audio/health.rs` is the accounting that makes a source's behaviour visible in the log, and **the two ends of the capture path are silent by design and must stay instrumented**: `capture::push_f32` discards samples when the ring is full and `mixer::pull_block` zero-pads when it is empty, neither of which can fail loudly — capture must never block and the mixer must never stall. So both return a count instead, the capture thread keeps a `CaptureStats` of atomics the engine reads, and `engine_loop` writes one `info` line per non-idle source every `HEALTH_INTERVAL` (30 s) plus one more from `stop_sources`. The line's fields are chosen to separate the three things that produce the same symptom: **ring occupancy climbing** across a session is the engine's wall clock outrunning the device's (padding inserts samples without consuming any, so a late packet adds latency that never drains — a one-way ratchet that ends with a full ring dropping samples mid-waveform); **discontinuities/gap frames** are Windows admitting it lost the audio before we saw it, read from the `BufferInfo` that `read_from_device_to_deque` returns; and the **measured ppm** is the device clock, taken from `BufferInfo.index` against a monotonic clock rather than from the format we asked for. Idle sources are skipped because a silent loopback source otherwise writes a line a minute in every user's log forever. `DeviceTimeline` handles the two ways `index` lies: the first packet has nothing to be differenced against, and a driver that does not report a position reports zero, which naively differenced makes the whole stream one enormous gap. Note that `BufferInfo.flags.silent` means the buffer contents are *undefined*, not zero, and the crate copies them out before we see the flag — `silence_appended` overwrites them, and removing it broadcasts uninitialized memory as noise.
3. **Network (tokio on a background thread)** — `src/net/`. `net_loop` owns the `AudioPubClient` (reqwest + cookie store) and, while streaming, two tasks: the Icecast source connection (hand-rolled `PUT` over `TcpStream` in `icecast.rs`) and the SSE consumer (`sse.rs` parser over `/live/{id}/events`).

TTS (`src/tts/`) is nine engines behind one router. `speaker::Speaker` holds two `tts::queue::Queue`s — bounded and **drop-oldest**, so a chat flood loses the stale messages, not the fresh ones — and dispatches on the engine id:

- **SAPI** (`tts/sapi.rs`) keeps its own COM STA thread owning `ISpVoice`, which synthesizes each message to memory at 48 kHz stereo and feeds `ExternalFeeds`. This thread's ownership rules are load-bearing; don't move work onto it. It used to *also* speak each message through an `ISpVoice` of its own, out of the default device and around the mixer — that is gone, and `tts::sapi::synth_preview` (its own apartment, samples handed back for `audio::cue`) is what the voice-preview button uses now that there is no local voice to borrow.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cha0t1cnu3tral/pubsplash](https://github.com/cha0t1cnu3tral/pubsplash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
