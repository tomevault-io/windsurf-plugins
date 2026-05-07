---
trigger: always_on
description: > Context document for GitHub Copilot.  Kept up-to-date so the assistant
---

# Copilot Instructions - Fancy Mumble

> Context document for GitHub Copilot.  Kept up-to-date so the assistant
> can skip expensive context-gathering on every request.

## Project overview

**Fancy Mumble** is a modern desktop Mumble (VoIP) client with profile
customisation features (avatar frames, banners, nameplates, effects).
Licensed MIT.  Written in Rust + TypeScript/React.

| Layer | Crate / package | Tech |
|-------|-----------------|------|
| Protocol library | `crates/mumble-protocol` | Rust, tokio, prost (protobuf), rustls, optional Opus codec |
| OMLSA denoiser | `crates/mumble-protocol` | Inlined in `src/audio/filter/denoiser/omlsa/` (Cohen 2001/2003), uses `realfft` |
| DeepFilterNet denoiser | `crates/fancy-denoiser-deepfilter` | Standalone Rust crate, `DeepFilterNet3` (Schroeter et al. 2023) via upstream `deep_filter` git dep + pinned `tract-onnx`/`ndarray` |
| Tauri backend | `crates/mumble-tauri` | Rust, Tauri 2, cpal (audio I/O, desktop only), rcgen (self-signed certs) |
| Tauri frontend | `crates/mumble-tauri/ui` | React 19, Vite 6, Zustand 5, react-router-dom 7, TypeScript 5 |
| Tauri Android | `crates/mumble-tauri/gen/android` | Gradle/Kotlin, Android API 34, NDK 27 |
| Dioxus GUI (alt) | `crates/mumble-gui` | Rust, Dioxus 0.7 desktop - older/parallel UI, same protocol lib |

## Workspace layout

```
Cargo.toml                          # Rust workspace root (resolver = "2")
ANDROID_DEV.md                      # Android development setup guide
scripts/
  android-dev.ps1                   # Android prerequisites checker & dev launcher (Windows)
crates/
  mumble-protocol/                  # Pure async Mumble client library
    proto/Mumble.proto              # TCP protobuf definitions
    proto/MumbleUDP.proto           # UDP protobuf definitions
    build.rs                        # prost-build code-gen
    src/
      lib.rs                        # re-exports: audio, client, command, error, event, message, proto, state, transport, work_queue
      client.rs                     # Async event-loop orchestrator (ClientConfig, ClientHandle, run())
      state.rs                      # ServerState: tracked users (User), channels (Channel), ConnectionInfo
      event.rs                      # EventHandler trait (on_control_message, on_udp_message, on_connected, on_disconnected)
      error.rs                      # Error enum (thiserror), Result alias
      message.rs                    # ControlMessage / UdpMessage enums, TcpMessageType id mapping
      work_queue.rs                 # Priority work queue: UDP > TCP > user commands
      command/
        mod.rs                      # CommandAction trait, CommandOutput, BoxedCommand
        authenticate.rs  join_channel.rs  send_audio.rs  send_text_message.rs
        set_comment.rs   set_texture.rs   set_self_mute.rs  set_self_deaf.rs
        set_voice_target.rs  channel_listen.rs  disconnect.rs  ban_user.rs
        kick_user.rs  request_blob.rs  request_ban_list.rs  request_user_stats.rs
        send_plugin_data.rs
      transport/
        tcp.rs                      # TcpTransport (TLS via tokio-rustls), TcpConfig
        udp.rs                      # UdpTransport, PlaintextCryptState, UdpConfig
        codec.rs                    # Wire framing helpers
        audio_codec.rs              # Opus-in-UDP framing
      audio/
        mod.rs                      # Pipeline architecture overview
        sample.rs                   # AudioFrame, AudioFormat
        capture.rs                  # AudioCapture trait + SilentCapture
        playback.rs                 # AudioPlayback trait + NullPlayback
        encoder.rs                  # AudioEncoder trait + OpusEncoder
        decoder.rs                  # AudioDecoder trait + OpusDecoder
        pipeline.rs                 # OutboundPipeline / InboundPipeline
        filter/                     # AudioFilter trait, FilterChain, NoiseGate, AutomaticGainControl
    tests/
      integration.rs               # End-to-end tests against a real murmur (via docker-compose)
      audio_quality.rs              # Audio pipeline quality tests

  mumble-tauri/                     # Tauri 2 desktop app
    tauri.conf.json                 # Product "Fancy Mumble", id com.fancymumble.app, 1024×768 frameless
    Cargo.toml                      # depends on mumble-protocol (with opus-codec), tauri 2, cpal, rcgen
    src/
      main.rs                       # Tauri entry point (calls lib::run)
      lib.rs                        # All #[tauri::command] handlers, app bootstrap
      audio.rs                      # CpalCapture / CpalPlayback - OS audio via cpal
      state/
        mod.rs                      # AppState struct, SharedState, query/messaging/channel/profile methods
        types.rs                    # UI value types (ChannelEntry, UserEntry, ChatMessage, etc.), event payloads, config structs
        connection.rs               # connect() / disconnect() lifecycle
        audio.rs                    # Voice pipeline management (enable, mute, deafen, outbound audio loop)
        event_handler.rs            # TauriEventHandler - EventHandler impl bridging protocol events to Tauri
    gen/
      android/                      # Generated Tauri Android project (Gradle/Kotlin, API 34)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fancy-Mumble/FancyMumbleNext](https://github.com/Fancy-Mumble/FancyMumbleNext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
