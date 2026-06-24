---
trigger: always_on
description: Hollow is a fully distributed, encrypted Discord alternative. No central servers. Members collectively host the server. See `HOLLOW_PLAN.md` for the full architecture, phase history, and current TODO checklist.
---

# HOLLOW — Project Instructions for Claude Code

## What Is This
Hollow is a fully distributed, encrypted Discord alternative. No central servers. Members collectively host the server. See `HOLLOW_PLAN.md` for the full architecture, phase history, and current TODO checklist.

## Tech Stack
- **UI:** Flutter (Dart) — all platforms (Windows, macOS, Linux, Android, iOS, Web)
- **Backend:** Rust via `flutter_rust_bridge` v2.11.1 FFI
- **Networking:** WSS relay (signaling + text/CRDT/MLS) + WebRTC data channels (files/shards P2P) + WebRTC media (voice/video P2P). libp2p fully removed.
- **E2EE:** vodozemac (Olm/Double Ratchet) for DMs, OpenMLS 0.8 for servers, SFrame (AES-128-GCM) for voice/video/screen share
- **Local DB:** SQLCipher (encrypted SQLite)
- **Identity:** Ed25519 keypairs via BIP-39 mnemonic (ed25519-dalek, NativeKeypair)
- **Org ID:** com.anonlisten
- **Project name:** hollow

## Project Structure
```
HOLLOW/
├── lib/                  # Dart/Flutter code (UI, app logic, state management)
│   ├── main.dart         # Entry point (ProviderScope + RustLib.init + window_manager init)
│   └── src/
│       ├── core/         # Models, Riverpod providers, service wrappers
│       ├── theme/        # Hollow design system (colors, spacing, typography, ThemeExtension)
│       └── ui/
│           ├── shell/    # Layout: hollow_shell, server_strip, channel_sidebar, member_panel, user_bar, mobile_nav, window_title_bar
│           ├── chat/     # ChatPane, MessageBubble, ChannelChatPane, ChannelMessageBubble
│           ├── settings/ # ServerSettingsPanel, OverviewTab, ChannelsTab, MembersTab, DangerZoneTab
│           ├── sidebar/  # PeerCard, EmptyPeerList
│           ├── components/ # HollowPressable, HollowButton, HollowTextField, HollowDialog, HollowTooltip, HollowToast, HollowToggle, HollowAvatar, HollowCard, StatusDot
│           ├── dialogs/  # InviteDialog, MnemonicDialog, CreateServerDialog, CreateChannelDialog, TwitchJoinDialog
│           └── animations/ # HollowCurves, HollowDurations, FadeSlideTransition, ScaleFadeTransition, SelectionShimmer, AmbientBackground, StartupRevealScope, RevealWidgets
├── rust/hollow_core/      # Rust library crate (networking, crypto, storage)
│   └── src/
│       ├── api/          # FFI layer (flutter_rust_bridge scans these)
│       ├── node/         # Networking modules (modularized from swarm.rs monolith)
│       │   ├── swarm.rs         # Event loop dispatcher + handle_incoming_request (~6.2k lines, envelope dispatch fully extracted)
│       │   ├── types.rs         # NetworkEvent, NodeCommand, HavenMessage, MessageEnvelope, helper structs
│       │   ├── crypto_handler.rs # Signing, Olm/MLS encryption, key exchange, coordinator election
│       │   ├── sync_handler.rs  # CRDT ops, server/channel CRUD, member management, sync
│       │   ├── message_ops.rs   # Send/edit/delete messages, emoji reactions (DMs + channels)
│       │   ├── social.rs        # Friends, profiles, typing indicators
│       │   ├── vault_ops.rs     # Vault shard storage, upload/download, recovery pool
│       │   ├── file_handler.rs  # File send/receive, stream handling, WebRTC transfers
│       │   ├── voice_handler.rs # Voice channels, 1:1 calls, WebRTC signaling
│       │   ├── gossip_relay.rs  # Gossip broadcast, peer exchange, timer handlers
│       │   ├── gossip.rs        # GossipOverlay, PeerScore, neighbor selection
│       │   ├── ws_client.rs     # WebSocket relay client
│       │   ├── ws_stream_transfer.rs # Binary stream reassembly
│       │   ├── signaling.rs     # Bootstrap peer discovery
│       │   ├── file_transfer.rs # File chunking utilities
│       │   ├── recovery_pool.rs # Recovery pool state management
│       │   ├── twitch.rs         # Twitch OAuth (Device Code Grant), follow/sub checks, proof validation
│       │   ├── image_convert.rs # WebP conversion
│       │   └── link_preview.rs  # URL link preview fetching
│       ├── crypto/       # Olm encryption + MLS + persistence
│       ├── identity/     # Ed25519 keypair management (native_identity.rs, keys.rs)
│       └── storage/      # SQLCipher message store
├── relay/                # Signaling HTTP + WS room router (Rust, legacy — superseded by relay-uws)
├── relay-uws/            # Production relay (uWebSockets C++, native TLS, deployed on OVH VPS)
├── packages/flutter_webrtc/ # Forked flutter_webrtc 1.4.1 (WASAPI loopback, native screen recording, macOS ScreenCaptureKit screen share)
├── rust_builder/         # flutter_rust_bridge build system (cargokit)
├── vendor/ffmpeg/        # Bundled native binaries (gitignored, see fetch_ffmpeg.ps1)
├── legal/                # Privacy Policy, Terms of Use, version manifest (manifest.json)
├── HOLLOW_PLAN.md         # Full architecture & design document (authoritative for all phase details)
└── CLAUDE.md             # This file
```

## Build & Run Commands
```bash
# Run on current platform (debug)
flutter run -d windows

# Build release
flutter build windows

# Run widget tests (no device needed, ~1s)
flutter test test/

# Check Rust code
cd rust/hollow_core && cargo check
cd rust/hollow_core && cargo clippy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VitalikPro13/HOLLOW](https://github.com/VitalikPro13/HOLLOW) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
