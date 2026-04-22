---
trigger: always_on
description: Cross-platform (iOS/macOS) SSH terminal app with iCloud sync and Keychain credential storage.
---

# VVTerm

Cross-platform (iOS/macOS) SSH terminal app with iCloud sync and Keychain credential storage.

## Target Versions

- **macOS**: 13.3+ (Ventura), arm64 only
- **iOS**: 16.1+, arm64 only
- **Xcode**: 16.0+

## Architecture

```
VVTerm/
├── App/
│   ├── VVTermApp.swift           # App entry point and composition root
│   ├── ContentView.swift         # Shared root container
│   ├── Localization/             # App-scoped localization preferences
│   └── iOS/                      # iOS app shell and root navigation views
├── Core/                         # Shared infrastructure and platform glue
│   ├── Logging/
│   ├── Network/
│   ├── UI/
│   ├── SSH/
│   ├── Security/
│   ├── Sync/
│   └── Terminal/
├── Features/                     # Feature-first product features
│   ├── ConnectionViews/
│   │   ├── Domain/
│   │   └── Application/
│   ├── LocalDiscovery/
│   │   ├── Domain/
│   │   ├── Application/
│   │   ├── Infrastructure/
│   │   └── UI/
│   ├── Servers/
│   │   ├── Domain/
│   │   ├── Application/
│   │   └── UI/
│   ├── RemoteFiles/
│   │   ├── Domain/
│   │   ├── Application/
│   │   ├── Infrastructure/
│   │   └── UI/
│   ├── VoiceInput/
│   │   ├── Infrastructure/
│   │   └── UI/
│   ├── Security/
│   │   ├── Domain/
│   │   ├── Application/
│   │   ├── Infrastructure/
│   │   └── UI/
│   ├── Settings/
│   │   ├── Application/
│   │   └── UI/
│   ├── Store/
│   │   ├── Domain/
│   │   ├── Application/
│   │   └── UI/
│   ├── Support/
│   │   └── UI/
│   ├── TerminalThemes/
│   │   ├── Domain/
│   │   ├── Application/
│   │   ├── Infrastructure/
│   │   └── UI/
│   ├── TerminalAccessories/
│   │   ├── Domain/
│   │   ├── Application/
│   │   └── UI/
│   ├── TerminalPresets/
│   │   ├── Domain/
│   │   ├── Application/
│   │   └── UI/
│   ├── TerminalSessions/
│   │   ├── Domain/
│   │   ├── Application/
│   │   └── UI/
│   ├── Stats/
│   │   ├── Domain/
│   │   ├── Application/
│   │   ├── Infrastructure/
│   │   └── UI/
│   └── Welcome/
│       ├── Domain/
│       └── UI/
├── GhosttyTerminal/              # libghostty terminal emulation
├── Compatibility/                # Version/platform compatibility helpers
├── Generated/                    # Build-time generated sources
└── Resources/                    # Bundled assets, themes, terminfo, l10n
```

## Architecture Direction

VVTerm uses a **feature-first architecture** for app-owned source code.

Current architecture:
- `App` owns app entry, composition roots, shared root containers, localization preferences, and iOS app-shell navigation.
- `Core/Sync` owns CloudKit sync infrastructure.
- `Core/Security` owns keychain, device identity, and privacy-mode infrastructure.
- `Core/Network` owns shared connectivity monitoring and Cloudflare transport support.
- `Core/UI` owns shared view primitives and presentation helpers reused across features.
- `Core/Terminal` owns shared clipboard, paste, and terminal text/default helpers.
- `Core/Logging` owns shared logging utilities.
- `Core/SSH` owns shared SSH bootstrap, known-hosts, key generation, environment detection, rich-paste support, tmux/mosh runtime helpers, and `SSHClient`.
- `Features/ConnectionViews` owns connection view tab configuration types and state.
- `Features/RemoteFiles` owns remote file browsing, preview, transfer, and SFTP integration.
- `Features/LocalDiscovery` owns discovery-specific code and UI.
- `Features/Servers` owns server/workspace domain models, server management, and server/workspace UI flows.
- `Features/Stats` owns server metrics collection and presentation.
- `Features/Security` owns app lock and biometric authentication flows.
- `Features/Settings` owns settings window presentation and settings screens.
- `Features/Store` owns Pro entitlements, purchases, and upgrade surfaces.
- `Features/Support` owns support/contact UI surfaces.
- `Features/TerminalThemes` owns theme models, validation, storage paths, parsing, and theme management.
- `Features/TerminalAccessories` owns keyboard accessory models, preferences, settings UI, and accessory validation flows.
- `Features/TerminalPresets` owns terminal preset models, persistence, and preset form UI.
- `Features/TerminalSessions` owns terminal session/tab domain models, session/tab managers, tmux prompt coordination, live activity support, and terminal session UI.
- `Features/VoiceInput` owns transcription/audio capture infrastructure, MLX model management, and transcription settings UI.
- `Features/Welcome` owns welcome/onboarding copy and presentation.
- New app code should land in `Features`, `Core`, or `App` based on ownership.
- New work inside a feature should stay inside its `Features/<FeatureName>` subtree and should not reintroduce app-wide bucket folders.

Feature-first shape:
- `Domain`: pure feature types and rules
- `Application`: feature state, orchestration, coordinators, use-case style logic
- `Infrastructure`: transport, persistence, adapters, external integrations
- `UI`: SwiftUI/AppKit/UIKit presentation only

For Files/SFTP specifically:
- no non-view logic under `UI`
- no feature policy inside `SSHClient` beyond low-level transport/session behavior
- use explicit dependency injection at the feature boundary
- do direct cutovers, not compatibility shims

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vivy-company/vvterm](https://github.com/vivy-company/vvterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
