---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Build both APKs (glasses APK is bundled into phone APK assets automatically)
./gradlew assembleDebug

# Build only glasses app
./gradlew :glasses-app:assembleDebug

# Build only phone app (triggers glasses build via bundleGlassesApk task)
./gradlew :phone-app:assembleDebug

# Clean build
./gradlew clean assembleDebug
```

The phone app's `preBuild` depends on a `bundleGlassesApk` task that copies `glasses-app-debug.apk` into `phone-app/src/main/assets/glasses-app-release.apk` for sideloading to glasses.

## Architecture

```
OpenClaw Gateway  ←WebSocket→  Phone App (Android)  ←Bluetooth/CXR→  Glasses App (Android)
     │                              │                                       │
  AI agent                     CXR-M SDK                               CXR-S SDK
  Chat streaming               OpenClawClient                          Chat HUD
  Sessions                     Voice input                             Gesture input
                               Bridge logic
```

Three Gradle modules:
- **shared/** — Protocol data classes (Gson-serialized). Used by both apps.
- **phone-app/** — Companion app. Connects to OpenClaw Gateway via WebSocket and to glasses via Rokid CXR-M SDK (Bluetooth) or debug WebSocket.
- **glasses-app/** — HUD app running on Rokid glasses. Receives messages from phone via CXR-S SDK bridge. Renders chat UI with Jetpack Compose.

## OpenClaw Protocol

[OpenClaw](https://openclaw.ai/) is an open-source personal AI assistant you run on your own devices, created by Peter Steinberger and the community ([source](https://github.com/openclaw/openclaw)). It supports multiple AI backends (Claude, GPT, local models). A local reference clone lives at `.openclaw-ref/` for protocol details.

The Gateway is OpenClaw's local control plane — it manages sessions, channels, tools, and events. Phone connects to the Gateway via `ws://host:port` (default 18789).

**Frame types:** Three discriminated JSON frames: `req` (`type`, `id`, `method`, `params?`), `res` (`type`, `id`, `ok`, `payload?`, `error?`), `event` (`type`, `event`, `payload?`, `seq?`, `stateVersion?`). `id` must be a string.

**Auth flow:** Gateway supports token or password auth modes (env: `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`). Our client uses token auth plus Ed25519 device identity: connect → receive `connect.challenge` event with nonce → send `connect` request with token + signed device payload → receive `hello-ok`. Device identity uses Android Keystore Ed25519 keypair (`DeviceIdentity.kt`).

**Chat streaming:** Client sends `chat.send` with `sessionKey`, `idempotencyKey`, `message`. Server responds with ack (`status: "started"`, `runId`), then pushes `chat` events with `state: "delta"|"final"|"error"`. Each delta contains **full accumulated text** (not incremental chunks) — client diffs against previous content to extract new text.

**Key connect params:** `client.id = "openclaw-control-ui"`, `client.mode = "ui"`, `scopes = ["operator.admin"]`. Origin header must be set on the WebSocket request.

## Phone ↔ Glasses Protocol

Defined in `shared/.../Protocol.kt`. JSON messages over CXR SDK (production) or WebSocket (debug).

**Phone → Glasses:** `chat_message`, `agent_thinking`, `chat_stream` (incremental chunk), `chat_stream_end`, `connection_update`, `session_list`, `voice_state`, `voice_result`

**Glasses → Phone:** `user_input` (text + optional imageBase64), `list_sessions`, `switch_session`, `slash_command`, `start_voice`, `cancel_voice`

## Glasses HUD

480x640 portrait display, JBD 0.13" micro-LED (~6,150 DPI), monochrome green on black. JetBrains Mono font. Font size auto-calculated to fit target column count based on display width.

**Layout:** TopBar (connection + mode indicator + scroll position) → ChatContentArea (LazyColumn) → MenuBar (Session | Size | Font | More)

**Two focus areas** — CONTENT and MENU. No INPUT focus (all input is voice via long-press).

| Area | Swipe Fwd | Swipe Bwd | Tap | Double-tap | Long-press |
|------|-----------|-----------|-----|------------|------------|
| CONTENT | Scroll up | Scroll down (push-through → MENU) | Jump to bottom | → MENU | Voice |
| MENU | Prev item (push-through → CONTENT) | Next item | Execute | → CONTENT | Voice |

**HUD position** cycles via SIZE menu: Full → Bottom Half → Top Half. SIZE icon dynamically shows what the next position will be.

**FONT** changes chat content font size by varying target columns: Compact(70) → Normal(60) → Comfortable(50) → Large(40). Menu bar uses fixed font size.

Voice results auto-submit immediately (no manual text entry).

## Rokid SDK Documentation

Full scraped SDK docs are available locally for reference:

- **CXR-M SDK (Mobile):** `docs/rokid-sdk/` — Bluetooth/WiFi connection, device controls, camera/audio, AI scene, teleprompter, translation, custom views
- **CXR-S SDK (Glasses):** `docs/rokid-sdk-glasses/` — On-device development, message subscription/sending, Caps data structure

Each directory has a `README.md` with an index. These docs were scraped from Rokid's custom documentation portal.

## SDK & Credentials


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dweddepohl/clawsses](https://github.com/dweddepohl/clawsses) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
