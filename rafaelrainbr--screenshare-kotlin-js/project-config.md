---
trigger: always_on
description: This file enables AI coding assistants to generate features aligned with this project's architecture and style. All patterns described here are derived exclusively from actual, observed conventions in the codebase — not invented best practices.
---

# GitHub Copilot Instructions — screenshare-kotlin-js

## Overview

This file enables AI coding assistants to generate features aligned with this project's architecture and style. All patterns described here are derived exclusively from actual, observed conventions in the codebase — not invented best practices.

> **Need more context?** Detailed analysis documents are available in [`docs/ai-analysis/`](../docs/ai-analysis/):
> - [`1-determine-techstack.md`](../docs/ai-analysis/1-determine-techstack.md) — full tech stack and domain boundaries
> - [`2-file-categorization.json`](../docs/ai-analysis/2-file-categorization.json) — every file mapped to its category
> - [`3-architectural-domains.json`](../docs/ai-analysis/3-architectural-domains.json) — architectural domains with required patterns and constraints
> - [`4-domains/`](../docs/ai-analysis/4-domains/) — deep-dive per domain (signaling, WebRTC, UI, media capture, etc.)
> - [`5-style-guides/`](../docs/ai-analysis/5-style-guides/) — per-category coding conventions

This project is a **real-time browser-based screen-sharing and voice-chat platform** built with Kotlin Multiplatform. A Ktor/JVM server provides WebSocket signaling; a Kotlin/JS client handles WebRTC, UI, and media capture.

---

## Tech Stack Summary

| Layer | Technology |
|---|---|
| Language | Kotlin 2.3.10 (JS + JVM, via KMP) |
| Server framework | Ktor 3.4.0 (CIO engine) |
| Client runtime | Kotlin/JS (Webpack, targets browser) |
| Serialization | kotlinx.serialization 1.10.0 |
| Async | kotlinx.coroutines 1.10.2 |
| UI styling | TailwindCSS + DaisyUI (CDN, `night` theme default) |
| Media | WebRTC browser APIs (`RTCPeerConnection`, `getDisplayMedia`, `getUserMedia`) |
| Tests | Kotest 6.1.3 (FunSpec, multiplatform) |
| Linting | KtLint 1.8.0 |
| Deployment | Ktor fat JAR → Docker → Fly.io (`gru` region) |

---

## File Category Reference

### `build-config`
_Gradle build files, version catalog, wrapper_

Examples: `build.gradle.kts`, `settings.gradle.kts`, `libs.versions.toml`, `client/build.gradle.kts`

Conventions:
- All versions in `libs.versions.toml` — never inline version strings in `.kts` files
- Plugins applied with `alias(libs.plugins.X)`; root-level plugins use `apply false`; subprojects apply via `subprojects { apply(plugin = "...") }`
- KMP source sets declare dependencies inside `kotlin { sourceSets { ... } }`
- The `copyClientToServer` task in `server/build.gradle.kts` embeds the Webpack bundle into the server JAR

### `entrypoints`
_Application entry points_

Examples: `client/src/jsMain/kotlin/Main.kt`, `server/src/jvmMain/kotlin/screenshare/server/Application.kt`

Conventions:
- Client `main()`: creates `WebsocketService` from `window.location`, creates `CoroutineScope(Dispatchers.Main + SupervisorJob())`, calls `registerUIHandlers()`
- Server: `fun main(args)` delegates to `EngineMain.main(args)`; module logic is in `fun Application.module()`

### `shared-protocol`
_Sealed `Packet` class — the wire protocol_

Examples: `common/src/commonMain/kotlin/screenshare/common/Packet.kt`

Conventions:
- Every message is a `data class` inside `sealed class Packet`
- All classes annotated `@Serializable @SerialName("kebab-case-type")`
- Short field `@SerialName`: `rid`=roomId, `sid`=socketId/senderId, `tid`=targetId, `msg`=message, `ice`=candidate
- Every packet classified as `CLIENT` or `SERVER` in `Packet.getSide()`

### `shared-models`
_Data transfer objects shared between client and server_

Examples: `common/src/commonMain/kotlin/screenshare/common/ChatMessage.kt`

Conventions:
- Flat `@Serializable data class`; no business logic
- Timestamps as `Long` (epoch ms)
- Defaults for optional state: `isMuted = true`

### `server-routing`
_Ktor routing and WebSocket endpoint setup_

Examples: `server/src/jvmMain/kotlin/screenshare/server/Application.kt`

Conventions:
- One WebSocket endpoint at `"/"`; `staticResources("/", "static")` for frontend
- `JoinRoom` handled directly in endpoint; all other packets routed to `Room.consumePacket()`
- Client IP resolved via `X-Forwarded-For` header with fallback

### `server-room-management`
_In-memory room and user lifecycle_

Examples: `server/src/jvmMain/kotlin/screenshare/server/Room.kt`

Conventions:
- Rooms created lazily with `computeIfAbsent`, removed when empty
- Chat history replayed to newly joined users
- After structural changes (join/leave): broadcast `UserConnected`/`UserDisconnected` then broadcast fresh `UserList`
- Logging via `LoggerFactory.getLogger(Room::class.java)`, format: `"Room [$id] ..."`

### `client-services`
_Stateful client service classes_

Examples: `services/Session.kt`, `services/VoiceChat.kt`, `services/ScreenSharing.kt`

Conventions:
- Plain classes, no DI framework; dependencies are constructor-injected
- `Session` is `CoroutineScope by coroutineScope`
- Action methods: `fun handleX() = launch { ... }` (return `Job`)
- Cross-service effects passed as lambda callbacks (`recreatePeerConnections: () -> Unit`), not direct references

### `client-websocket`
_WebSocket connection management_


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RafaelrainBR/screenshare-kotlin-js](https://github.com/RafaelrainBR/screenshare-kotlin-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
