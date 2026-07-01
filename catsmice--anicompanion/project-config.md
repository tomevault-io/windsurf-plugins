---
trigger: always_on
description: macOS desktop AI character app with VRM 3D character rendering, LLM chat, TTS speech, and STT voice input.
---

# AniCompanion

macOS desktop AI character app with VRM 3D character rendering, LLM chat, TTS speech, and STT voice input.

> The character (小光) talks to you, speaks with a synthesized voice, lip-syncs and emotes a
> VRM avatar, and can proactively start conversations. The LLM runs through a local
> **[Hermes Agent](https://github.com/NousResearch/hermes-agent)** gateway you run yourself
> ("bring your own Hermes"). See `README.md` for setup.

## Tech Stack

- **Language**: Swift 6.0, macOS 15.0+
- **UI**: SwiftUI
- **Character rendering**: VRM via three-vrm (pixiv) in WKWebView + WebGL
- **LLM**: Local **Hermes Agent** gateway — OpenAI-compatible HTTP API (`POST
  http://127.0.0.1:8642/v1/chat/completions`, `stream:true` SSE, Bearer auth). Bring your own.
- **TTS**: MiniMax Speech-02-Turbo (streaming SSE, hex-encoded MP3) — user supplies the API key
- **STT**: Apple Speech Framework (on-device)
- **Audio**: AVAudioEngine + AVAudioPlayerNode
- **Dependencies**: no SPM packages — three-vrm + three.js load via CDN import maps; networking via
  URLSession, audio via AVFoundation. See `ATTRIBUTION.md`.

## Project Structure

```
AniCompanion/
├── AniCompanion/
│   ├── App/             # App entry, AppState, Configuration
│   ├── Views/           # SwiftUI views (Main, Chat, Settings)
│   ├── Services/        # ChatTransport + HTTPChatService (Hermes), TTS, STT, AudioPlayer, ObjCSupport
│   ├── Character/       # ThreeVRMCharacterManager, ThreeVRMRenderView (WKWebView bridge to three-vrm)
│   ├── Pipeline/        # Orchestration (ConversationController, SentenceParser, AudioQueue)
│   ├── Models/          # Data models (ChatMessage, Emotion, ConversationHistory, AnimationClip)
│   └── Resources/       # Persona/<lang>/ (system_prompt.txt + proactive.json), Localizable.xcstrings, VRMModel/, Animations/ (JSON keyframe clips), ThreeVRM/ (HTML+JS scene)
├── Tools/               # Blender animation export + app-icon generator
├── scripts/             # download-model.sh (fetches the default VRM)
├── project.yml          # XcodeGen project spec
└── AniCompanion.xcodeproj
```

## Key Commands

```bash
# Regenerate Xcode project after changing project.yml (run from repo root, where project.yml lives)
xcodegen generate

# Build
xcodebuild -project AniCompanion.xcodeproj -scheme AniCompanion -destination 'platform=macOS' build

# Fetch the default VRM model (not committed — see ATTRIBUTION.md)
./scripts/download-model.sh

# Open in Xcode
open AniCompanion.xcodeproj
```

## Architecture

### Streaming Pipeline (critical path)

```
User input (text or voice) → HTTP chat (Hermes) → SentenceParser → parallel TTS → AudioQueue → ordered playback + lip sync
```

- **ChatTransport** (protocol) + **HTTPChatService** (@MainActor): `POST /v1/chat/completions`
  with `stream:true`; parses OpenAI-standard SSE (`data: {json}` lines, `data: [DONE]` terminator)
  into an `AsyncStream<WSIncoming>` of token/done/error events. `connect()` = `GET /health` (no
  persistent socket, heartbeat, or reconnect). Cancel → URLSession task cancel.
- **ChatBackend** (enum registry): "bring your own agent" seam. Each `case` registers a backend
  (`displayName`/`defaultEndpoint`/`defaultModel`/`configHint` + a `makeTransport(_:)` arm returning
  an `any ChatTransport`). `AppState` builds `ChatBackend.current.makeTransport(BackendConfig(...))`;
  the Settings **Agent backend** picker lists all `allCases`. Adding a backend = implement a
  transport + add a case. The selected backend is stored under `chat_backend`; each backend persists
  its **own** endpoint + key under per-backend keys (`chat_endpoint_<rawValue>` /
  `chat_api_key_<rawValue>`) via `savedEndpoint()`/`savedAPIKey()`/`saveConnection()`. A one-time
  migration (`migrateLegacyConnectionDefaults`) folds the older single-connection keys
  (`chat_endpoint`/`chat_api_key`, and before that `hermes_*`) into Hermes' per-backend keys.
  See `CONTRIBUTING.md` → "Adding an agent backend".
- **SentenceParser** (actor): Buffers LLM chunks, detects Chinese sentence boundaries, extracts emotion tags
- **AudioQueue** (actor): FIFO queue ensuring ordered playback even when TTS responses arrive out of order
- **ConversationController** (@MainActor): Orchestrates the full pipeline; consumes transport events
  via a long-lived listener; bridge-stream pattern (tokens → AsyncThrowingStream continuation);
  60-min proactive idle timer. (A dormant `notify`/`runNotificationPipeline`/`ack` path remains for a
  future cron-push integration; nothing emits `notify` today.)

### VRM Character Rendering (three-vrm + WKWebView)

- **ThreeVRMCharacterManager** (@MainActor): Swift bridge implementing `CharacterControllerProtocol`,
  sends commands to JS via `WKWebView.evaluateJavaScript()`, receives events via `WKScriptMessageHandler`
- **ThreeVRMRenderView**: SwiftUI `NSViewRepresentable` wrapping WKWebView, keyboard camera controls (W/S/A/D/Q/E/R/F)
- **vrm_scene.js**: three.js + @pixiv/three-vrm scene; WebGL rendering, idle animations, animation player, expression/lip-sync control
- **Spring bones**: Work via three-vrm's `vrm.update(delta)` — hair/skirt physics with gravity and colliders

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [catsmice/AniCompanion](https://github.com/catsmice/AniCompanion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
