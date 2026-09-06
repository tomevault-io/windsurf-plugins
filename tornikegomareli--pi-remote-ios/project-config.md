---
trigger: always_on
description: This file is the entry point for any agent working on `pi-remote-ios`. Read it first. For deeper specs see `Systems.md`, `PLAN.md`, and `docs/`. `CLAUDE.md` is a symlink to this file, so Claude Code and other agents read identical instructions — edit only `AGENTS.md`.
---

# AGENTS.md — Project orientation

This file is the entry point for any agent working on `pi-remote-ios`. Read it first. For deeper specs see `Systems.md`, `PLAN.md`, and `docs/`. `CLAUDE.md` is a symlink to this file, so Claude Code and other agents read identical instructions — edit only `AGENTS.md`.

## What this project is

A two-half system that lets a paired iOS app control an active Pi (`@earendil-works/pi-coding-agent`) terminal session over WebSocket:

1. **TypeScript Pi extension** (`extension/`) — loaded into a running `pi` process via the project's `.pi/settings.json`. Exposes a `/remote` slash command, runs an HTTP/WebSocket server, owns auth + protocol + Pi event mapping.
2. **iOS app** (`Pi/`) — native SwiftUI app built on **The Composable Architecture (TCA)**. Connects to the extension over WebSocket, pairs once with a 6-digit code, then drives the session: send prompts, abort, compact, observe streaming assistant output and tool calls.

User flow:

```
$ pi                           # in repo root, loads extension via .pi/settings.json
> /remote start                # starts ws server on the Pi host's local network.
                               # Prints status, 6-digit code, and a QR code encoding a
                               # pi-remote://pair?url=...&code=... deep link.

iPhone: scan QR from Camera or in-app → auto-fills + auto-pairs
                               # extension returns one-time device token
                               # iOS stores the token in Keychain for future reconnects

iPhone: type prompt → Send     # round-trip through WebSocket → Pi → LLM → streaming back
                               # requires the iPhone and Pi host on the same trusted network
```

`/remote start` means "running and ready to pair." There is intentionally no separate `/remote pair` command.

## Repository layout

```
pi-remote-ios/
├── Project.swift           Tuist source of truth for iOS targets and settings
├── Tuist.swift             shared Tuist configuration
├── Tuist/Package.swift     external Swift package dependencies
├── extension/              TypeScript Pi extension (server side)
│   ├── index.ts            thin entry point
│   ├── command.ts          /remote command handler + UI notifications
│   ├── config.ts           defaults + env overrides
│   ├── runtime/            RemoteControlRuntime (start/stop/status/rotate)
│   ├── server/             HTTP + WebSocket upgrade lifecycle
│   ├── connections/        RemoteConnection (socket lifecycle) + RemoteSession (auth + commands)
│   ├── auth/               DeviceTrust + PairingManager + TokenStore + crypto
│   ├── protocol/           wire envelopes + ProtocolDecoder + capabilities + error codes
│   ├── pi/                 PiRemoteAdapter (seam around ExtensionAPI) + sanitizer + editPayload (unified-diff extraction/synthesis)
│   ├── events/             RemoteEventStream (Pi event subscription + broadcast)
│   ├── questions/          remote_question tool + reconnect-safe QuestionBroker
│   ├── session/            structured phase monitor + run evidence collector
│   ├── tunnel/             dormant legacy Cloudflare tunnel implementation
│   └── logging/            structured logger
│
├── test/                   Node test suite for the extension (decoder/auth/integration)
│
├── docs/
│   ├── PROTOCOL.md         full wire protocol spec
│   ├── SECURITY.md         threat model + security invariants
│   ├── SWIFT_CLIENT_NOTES.md  iOS integration notes
│   └── API.md              short index
│
├── Pi/                     iOS app sources and tests
│   ├── PiTests/            Swift Testing coverage for pure app policies
│   ├── PiUITests/          XCTest UI coverage for live release acceptance
│   └── Pi/                 app sources (auto-included via synced root group)
│       ├── PiApp.swift     @main, instantiates the AppFeature Store
│       ├── DesignSystem/   AppColors / AppFonts / AppSpacing — single source of truth
│       ├── Components/     reusable views and the native Canvas Pi logo animation
│       ├── Core/
│       │   ├── Background/ ContinuedProcessingClient for minimized run monitoring
│       │   ├── Networking/ RemoteSocketClient (hand-rolled dependency; see comment)
│       │   ├── Notifications/ privacy-safe local completion/question alerts
│       │   ├── Persistence/CredentialsStore (Keychain-backed token + URL + device)
│       │   ├── Protocol/   wire-format types (mirror of extension/protocol)
│       │   ├── Model/      domain types (transcript, state snapshot, PairingDeepLink)
│       │   └── Extensions/ binding helpers
│       └── Features/
│           ├── AppFeature.swift       single TCA reducer that owns the session domain
│           ├── AppRootView.swift      routes by AppFeature.State.screen
│           ├── Pairing/               PairingView + PairingScannerSheet
│           ├── Dashboard/             DashboardView + DashboardHeader + Composer + Transcript/
│           ├── Reconnect/             ReconnectView
│           └── Settings/              SessionSettingsView + EnvelopeLogRow
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tornikegomareli/pi-remote-ios](https://github.com/tornikegomareli/pi-remote-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
