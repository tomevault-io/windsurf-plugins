---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

"KALA" (package name `terminal_agent`) is a Flutter app that combines an SSH connection manager, a multi-session terminal emulator (SSH), a remote file explorer (SFTP), and a code editor into a single mobile-first IDE-like tool. Configured platforms are **Android** and **Linux** only.

## Commands

Two dependencies are vendored under `third_party/` and wired through `dependency_overrides` in `pubspec.yaml`: the patched `xterm`, and `dartssh2` — whose patch fixes an upstream bug where a channel's receive window is never replenished once exhausted, deadlocking any transfer over ~6 MB through a tunnel. Re-apply both patches if either package is upgraded.

The Flutter SDK is vendored inside this repo at `sdk/flutter` (a full flutter/flutter checkout used as the project's Dart/Flutter SDK — it is not part of the app's own source and generally doesn't need to be touched). If `flutter` is not on `PATH`, use `sdk/flutter/bin/flutter`.

- Install dependencies: `flutter pub get`
- Run the app (Linux desktop): `flutter run -d linux`
- Run the app (Android): `flutter run -d <android-device-id>`
- Static analysis / lints: `flutter analyze`
- Run all tests: `flutter test`
- Run a single test file: `flutter test test/widget_test.dart`
- Build Linux release: `flutter build linux`
- Build Android APK: `flutter build apk`

Note: `test/widget_test.dart` is still the default Flutter counter-app template and does not match `MyApp` (no counter UI exists), so `flutter test` currently fails. Replace or rewrite this test before relying on it.

## Architecture

### State management

All app state is centralized in a single `ChangeNotifier`, `AppState` (`lib/providers/app_state.dart`), provided at the root via `provider` in `lib/main.dart`. Views read it with `Provider.of<AppState>(context)`. There is no other state management layer — new features should extend `AppState` rather than introducing local widget state for anything that needs to survive tab switches.

### Multi-session terminal model

- `AppState` holds a list of `TerminalSession` objects (`_sessions`), each with its own `xterm.Terminal`, `ConnectionStatus` (`disconnected | connecting | remote`), and a `dartssh2.SSHClient`/`SSHSession` (remote shell).
- Only one session is "active" at a time (`_activeSessionIndex`). Most getters (`terminal`, `connectionStatus`, `currentPath`, `files`, etc.) are convenience delegates to `activeSession`.
- Connecting to a saved profile (`connectToSSH`) creates a *new* session and connects it via `_connectSessionToSSH`, then switches the active tab to the terminal.
- `disconnect` / session loss tears down the session's SSH connection and marks it as disconnected.

### Connection profiles

`ConnectionProfile` (`lib/models/connection_profile.dart`) is a plain JSON-serializable model (host/port/username/password/etc.). Profiles are persisted as a JSON string list under the `ssh_profiles` key via `shared_preferences` (`_loadProfiles`/`saveProfile`/`deleteProfile` in `AppState`). Passwords are stored in plaintext in shared preferences — there is no keychain/secure-storage integration.

### Port forwarding (tunnels)

Tunnels are configured per profile (`ConnectionProfile.tunnels`, a list of `SshTunnel` — see `lib/models/ssh_tunnel.dart`) and run by `TunnelManager` (`lib/services/tunnel_manager.dart`), a separate `ChangeNotifier` owned by `AppState` and provided alongside it in `main.dart` so live byte counters don't rebuild the whole app.

- All three OpenSSH kinds are supported: `-L` (local), `-D` (SOCKS5, implemented by dartssh2's `forwardDynamic`) and `-R` (remote).
- Tunnels are tied to their session: started in `_connectSessionToSSH` via `syncOnConnect`, stopped on connection loss (`onSessionLost`, which keeps `TunnelRuntime.desired` so a reconnect restores exactly what was up) and released in `_cleanupSession` (`removeSession`).
- `saveProfile` calls `syncConfig` on live sessions, so adding/editing a tunnel applies without reconnecting.
- Legacy `PortForward`/`forwards` JSON is migrated into `tunnels` on load, and `toMap` still mirrors local tunnels into `forwards` so a downgrade doesn't lose them.
- Listening sockets bind to loopback unless `SshTunnel.exposeToLan` is set; ports below 1024 are rejected up front (Android can't bind them).
- `SshTunnel.idleTimeoutMinutes` closes a tunnel after N minutes with **zero open connections** — it narrows the window in which another app on the phone can reach the local port, and never cuts a session in use (the countdown is armed/cancelled from `_onConnectionCountChanged`). Not offered for SOCKS, whose connections dartssh2 doesn't report.
- UI: `lib/views/tunnels_tab.dart` (tab index 9, reachable from the drawer), a badge + sheet in the terminal toolbar, and the shared editor `lib/views/tunnel_editor_sheet.dart` used by both the profile form and the console.

### Host key verification


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jhongdlp/Kammel_ssh](https://github.com/Jhongdlp/Kammel_ssh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
