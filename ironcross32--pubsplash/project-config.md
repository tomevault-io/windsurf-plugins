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
- Ignored tests (hit the real audiopub.site or play audio): `cargo test <name> -- --include-ignored`. Notable: `real_login_bad_credentials` (live server), `sapi_speaks` (audible), `sapi_synthesizes_pcm`
- Run: `./target/debug/pubsplash.exe` (release builds hide the console via `windows_subsystem`)

Per `agents.md`: every change set must update `changelog.md` (bulleted entries under `## Unreleased` → `### Additions`/`### Fixes`/`### Changes`) and keep `README.md` accurate (especially the shortcut table).

## Architecture

Three long-lived domains connected by channels; the UI never blocks on audio or network:

1. **UI (main thread)** — `src/ui/`. wxdragon main loop. A 100 ms `Timer` ("the pump", in `ui/mod.rs::pump_events`) drains `NetEvent`/`EngineEvent` receivers into UI state; once a second it refreshes durations and chat relative times. `App` (an `Rc` holding `RefCell`s) is the shared UI-side state: persisted `Config`, transient `Runtime`, handles to the engine/net/speaker threads, and `Widgets` (populated after `ui::build`).
2. **Audio engine (thread)** — `src/audio/`. `engine_loop` mixes 10 ms blocks: per-source WASAPI capture threads push f32 into rtrb rings → per-source `ChannelStrip` (volume + 50 ms mute fades) → master strip → optional LAME MP3 encode → tokio mpsc to the Icecast sender. Sources that aren't OS-captured (TTS, sound events) are `FeedKind::External`: the engine parks an rtrb producer in `ExternalFeeds` keyed by source name, and other subsystems push samples via `ExternalFeeds::push` (retry on `Full`, abort on `Gone`).
3. **Network (tokio on a background thread)** — `src/net/`. `net_loop` owns the `AudioPubClient` (reqwest + cookie store) and, while streaming, two tasks: the Icecast source connection (hand-rolled `PUT` over `TcpStream` in `icecast.rs`) and the SSE consumer (`sse.rs` parser over `/live/{id}/events`).

TTS (`src/tts/sapi.rs`) runs on its own COM STA thread owning `ISpVoice`: requests arrive over a channel; each chat message is spoken locally (async) and, when `output_to_stream`, synthesized to memory at 48 kHz stereo and trickle-fed into the engine via `ExternalFeeds`.

`Config` (in `config.rs`) is both the persisted settings file and the scene/source model; `state.rs` adds list-manipulation methods (`switch_to`, `delete_scene`, `move_up`...) that return `ListEdit::Changed/Unchanged` so the UI knows whether to refresh/save. After any scene/source edit, call the chain in `ui/scenes.rs::after_scene_edit` (save → refresh scenes list → `on_sources_changed` → `refresh_app_processes` + `sync_engine_sources` + `rebuild_mixer` → refresh sources list, which must come last so it reads the re-resolved app cache).

Everything a user sees or hears a source called comes from `src/source_name.rs`, never from `SourceConfig.name` — that field is an identity key (it routes `ExternalFeeds` and TTS `SpeakRequest`s) and is always just the kind's display name plus a counter, since there is no rename UI. `list_labels` is the verbose Scenes-list form, `strip_labels` the concise mixer form; both dedupe within a scene. They read a `NameContext` (capture devices + resolved Application processes) built from `App::name_context`. Application sources resolve to a running process via `audio::device::resolve_apps`, cached in `Runtime.apps` and refreshed every 2 s by the pump; when a pid changes the pump re-syncs the engine, and when a name changes it calls `home::relabel_source_strips`, which re-labels strips **in place** (a `rebuild_mixer` would move focus).

## Hard-won protocol facts (verified against the real server; do not "simplify" these away)

- The server kills any live source that isn't **MP3 or AAC** (it ffprobes the mount). MP3 via LAME is the default; the spec's mention of Opus is unimplementable.
- SvelteKit form actions (`/login`, `/live/new`) return **200 + JSON envelope** (`{"type":"redirect"|"failure",...}`) to non-browser clients, not a real 303. Parse the envelope (`audiopub.rs::action_result`), never trust the HTTP status alone.
- Chat SSE user objects contain **both `name` and `displayName`** — they must be separate serde fields (an alias makes serde reject the payload as a duplicate field, silently dropping every message).
- `__data.json` responses use SvelteKit's devalue flat-array encoding, and multiple nodes can carry a `user` object; only the stream-instructions page node has `streamKey` — skip non-matching nodes, don't bail.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ironcross32/pubsplash](https://github.com/ironcross32/pubsplash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
