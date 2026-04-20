---
trigger: always_on
description: This file is a routing layer for coding agents working in this repo. Keep it short. Put long-lived detail in nearby code, focused docs, or tests.
---

# AGENTS.md

This file is a routing layer for coding agents working in this repo. Keep it short. Put long-lived detail in nearby code, focused docs, or tests.

## Mission

- `PingIsland` is a macOS menu bar app that surfaces Dynamic Island-style status for Claude Code, Codex, Gemini CLI, Hermes Agent, Qwen Code, and compatible hook-driven agent sessions.
- The main runtime path is:
  - hook or app-server events
  - monitoring and service layers
  - `SessionStore`
  - `SessionMonitor` and `NotchViewModel`
  - SwiftUI notch UI
- There are two important codepaths:
  - `PingIsland/`: the shipping Xcode app
  - `Prototype/`: a SwiftPM prototype with focused tests and reference implementations

## Start Here

- Product overview: `README.md`
- App entry: `PingIsland/App/PingIslandApp.swift`, `PingIsland/App/AppDelegate.swift`
- Main state hub: `PingIsland/Services/State/SessionStore.swift`
- Session association cache: `PingIsland/Services/State/SessionAssociationStore.swift`
- Native runtime rollout scaffold: `PingIsland/Services/Runtime/`, `PingIsland/Core/FeatureFlags.swift`
- Session bridge for UI: `PingIsland/Services/Session/SessionMonitor.swift`
- Notch state and layout: `PingIsland/Core/NotchViewModel.swift`, `PingIsland/UI/Views/NotchView.swift`
- Global shortcuts and shortcut persistence: `PingIsland/Services/Shared/GlobalShortcutManager.swift`, `PingIsland/Utilities/GlobalShortcut.swift`, `PingIsland/Core/Settings.swift`, `PingIsland/UI/Views/SettingsWindowView.swift`
- Claude hook ingress: `Prototype/Sources/IslandBridge/`, `PingIsland/Services/Hooks/HookInstaller.swift`, `PingIsland/Services/Hooks/HookSocketServer.swift`
  - `PingIslandBridge` is the unified Claude/Codex hook entrypoint and is responsible for terminal, tmux, SSH-remote, and IDE terminal context capture before envelopes hit Swift code
- Codex ingress: `PingIsland/Services/Codex/`, `PingIsland/UI/Views/CodexSessionView.swift`
  - Hook-less fallback parsing for Codex sessions lives in `PingIsland/Services/Codex/CodexRolloutParser.swift`
- Terminal and focus control: `PingIsland/Services/Tmux/`, `PingIsland/Services/Window/`, `PingIsland/Utilities/TerminalVisibilityDetector.swift`
  - Terminal focus flows currently cover iTerm2, Ghostty, Terminal.app, tmux, and IDE-hosted terminals
- Remote SSH forwarding and remote-host management: `PingIsland/Services/Remote/`
  - Remote hosts can bootstrap a bridge on the SSH target, rewrite remote hooks, install managed plugin-directory integrations such as Hermes under the remote home directory, and attach a bidirectional forwarding channel back into PingIsland
- Provider/client routing: bridge envelopes are normalized in `PingIsland/Services/Hooks/HookSocketServer.swift`, stored on `SessionState`, and launched via `PingIsland/Services/Window/SessionLauncher.swift`
- Client profile registry: installable hook clients and runtime client branding / recognition are centralized in `PingIsland/Models/ClientProfile.swift`
- VS Code-compatible IDE focus extension install / URI launch: `PingIsland/Services/Window/IDEExtensionInstaller.swift`, `PingIsland/Services/Window/TerminalSessionFocuser.swift`
- Session list UI: `PingIsland/UI/Views/SessionListView.swift`
- Client mascot system: `PingIsland/UI/Components/MascotView.swift`, `PingIsland/UI/Views/MascotSettingsView.swift`
- App updates and release notes: `PingIsland/Services/Update/`, `PingIsland/UI/Views/ReleaseNotesWindowView.swift`, `PingIsland/UI/Window/ReleaseNotesWindowController.swift`
- Sparkle build configuration: `Config/App.xcconfig`, `Config/LocalSecrets.xcconfig`, `docs/sparkle-release.md`

## Repo Map

- `PingIsland/App`: app lifecycle, window setup, screen observation
- `PingIsland/Core`: notch geometry, shared state, app settings, selectors
- `PingIsland/Models`: domain models for sessions, events, tools, phases
- `PingIsland/Services`: ingestion, socket handling, state management, tmux, windows, updates
- `PingIsland/Services/Runtime`: isolated native Claude/Codex runtime work. This path should coexist with the current implementation behind feature flags until parity is proven.
- `PingIsland/Services/Remote`: remote endpoint persistence, SSH bootstrap / attach, and remote hook forwarding
  - Remote bootstrap currently covers JSON hook configs, managed hook directories, and managed plugin directories (for example remote Hermes installs under `~/.hermes/plugins/ping_island`)
- `PingIsland/Services/Update`: Sparkle updater bridge, appcast/release-notes loading, update state publishing
- `PingIsland/Services/Window/IDEExtensionInstaller.swift`: installs the VS Code-compatible terminal-focus extension used by Cursor / VS Code / CodeBuddy / Qoder style IDE hosts (`QoderWork` is hook-only, not an IDE extension host)
- `PingIsland/UI`: SwiftUI views, reusable components, AppKit-backed window controllers
- `PingIsland/Resources`: hook assets, entitlements, bundled fonts
- `Prototype`: Swift package prototype and testbed
- `Prototype/Tests`: logic-level unit tests plus process/socket e2e coverage for `IslandBridge`, hook mapping, and install flows
- `scripts`: release, signing, and packaging automation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erha19/ping-island](https://github.com/erha19/ping-island) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
