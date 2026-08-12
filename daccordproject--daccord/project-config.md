---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

This repository is a **Flutter/Dart client for [Daccord](https://github.com/DaccordProject)** — a free, open-source chat platform for communities. It is a **fork of [Bonfire](https://github.com/OpenBonfire/bonfire)**, a fast cross-platform Discord client written in Flutter.

We are repurposing Bonfire's mature, well-structured Flutter UI and reusing as much of it as possible, while **replacing its Discord networking layer with the Accord protocol**. The goal is a native, multi-platform Daccord client that reaches feature parity with the existing Godot-based [`daccord`](https://github.com/DaccordProject/daccord) client.

### Hard requirements (do not violate)

- **No Discord integration.** This client talks **only** to Daccord/Accord servers. All `discord.com`, `cdn.discordapp.com`, Discord gateway, Discord OAuth/token, and Firebase-push code paths are to be removed or replaced. Do not add Discord endpoints back.
- **License stays GPLv3.** Bonfire is licensed GPL-3.0 and we retain it. See `LICENSE`. AccordKit-Dart and the Godot daccord client are MIT — GPLv3 may incorporate MIT-licensed code, so depending on `accordkit` is fine. Keep the `LICENSE` file as GPL-3.0; any new files inherit GPLv3.
- **Reuse Bonfire.** Prefer adapting existing Bonfire widgets, controllers, routing, theming, and caching over rewriting. The networking/models swap is the bulk of the work; the UI should change as little as possible.
- **Voice, video & screen sharing are implemented.** Real-time voice, video, and screen sharing (LiveKit/WebRTC transport) are fully supported. Maintain and extend the existing voice stack; don't stub or hide voice UI.

## The three repositories involved

| Repo | What it is | Language | License | Role here |
|------|-----------|----------|---------|-----------|
| **this repo** (was `bonfire`) | Flutter UI we're adapting | Dart/Flutter | GPL-3.0 | The client we ship |
| [`daccord`](https://github.com/DaccordProject/daccord) (`../daccord`) | Existing reference client | GDScript / Godot 4.5 | MIT | Feature/UX reference to match |
| [`accordkit-dart`](https://github.com/DaccordProject/accordkit-dart) (`../accordkit-dart`) | Accord protocol SDK | Dart | MIT | Our networking layer (replaces firebridge) |

The Accord server backend is [`accordserver`](https://github.com/DaccordProject/accordserver) (Rust). The protocol is documented via [`accordserver-mcp`](https://github.com/DaccordProject/accordserver-mcp).

## Architecture (inherited from Bonfire)

- **State management:** Riverpod 3 (`flutter_riverpod`, `riverpod_annotation` with codegen → `*.g.dart`).
- **Models / serialization:** primarily provided by `accordkit` (`Accord*` types). The handful of client-local models (server config, session, device profile, space folders, settings) hand-roll `fromJson`/`toJson` — they're small and Hive-backed, so codegen serializers aren't used. `freezed`/`json_serializable` remain (dev) dependencies but are unused by any model: no `*.freezed.dart` or model `*.g.dart` files exist in `lib/` (the only generated files are Riverpod's). They can be dropped once someone regenerates `pubspec.lock` locally.
- **Routing:** `go_router`.
- **Local storage:** `hive_ce` — boxes opened in `setupHive()`: `auth`, `last-location`, `added-accounts`, `accord-session`, `accord-settings`.
- **Networking:** `accordkit` (vendored in-tree at `packages/accordkit`, maintained here). **The firebridge → accordkit swap is complete** — `packages/firebridge` and `firebridge_extensions` no longer exist and nothing in `lib/` imports them (a few doc comments still mention "firebridge" to describe what a controller replaced). Do not try to re-add firebridge.
- **Voice/video/screen share:** `livekit_client` (a local fork at `packages/livekit_client`, see #68) over WebRTC; credentials fetched via accordkit's `client.voice`. See `lib/features/voice/`.
- **Media:** `media_kit` (+ `media_kit_video`, `video_player_media_kit`, pinned git forks) / `cached_network_image` / `file_picker` — re-point CDN URLs at the Accord server.
- **Code generation is required during development:** `dart run build_runner watch -d`.

### Layout

```
lib/
  features/        # feature modules: authentication, spaces, channels, messaging,
    <feature>/     #   member, user, admin, server, events, voice, notifications,
      controllers/ #   settings, developer, profiles, updates, ...
      repositories/# data access (accordkit-backed)
      views/       # screens
      models/      # feature models
  shared/          # shared components, models, repositories, utils
  theme/           # theming
  router/          # go_router config
  main.dart        # startup: media_kit, Hive, ProviderScope, ProfileGate, deep links
packages/
  accordkit/       # Accord protocol SDK (REST + gateway + models) — networking layer, maintained here
  livekit_client/  # local fork of livekit_client 2.8.0 (#68 native-release fix) — voice transport
  markdown_viewer/ # custom markdown rendering — protocol-agnostic, KEEP
docs/              # product + technical specs (see below)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DaccordProject/daccord](https://github.com/DaccordProject/daccord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
