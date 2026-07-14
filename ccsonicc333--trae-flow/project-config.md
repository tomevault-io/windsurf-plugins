---
trigger: always_on
description: This file is a routing layer for coding agents working in this repo. Keep it short. Put long-lived detail in nearby code, focused docs, or tests.
---

# AGENTS.md

This file is a routing layer for coding agents working in this repo. Keep it short. Put long-lived detail in nearby code, focused docs, or tests.

## Mission

- TRAE FLOW is a macOS menu bar app that surfaces Dynamic Island-style status for four TRAE variants: TRAE, TRAE CN, TRAE WORK, and TRAE WORK CN.
- Scope: only the four TRAE variants are supported. Non-TRAE AI clients (Claude, Codex, Gemini, Hermes, Qwen, OpenClaw, CodeBuddy, OpenCode, Kimi, Pi, Qoder, Copilot) and their logos, mascots, hook integrations, native runtime, usage dashboards, remote SSH forwarding, and IDE extension installer have been removed.
- The main runtime path is:
  - Trae official hook events (SessionStart / UserPromptSubmit / PreToolUse / PostToolUse / Stop / Notification)
  - `TraeFlowBridge` with `--variant` argument
  - monitoring and service layers
  - `SessionStore`
  - `SessionMonitor` and `NotchViewModel`
  - SwiftUI Flow Island UI (left: custom HTML / session detail, right: variant counts / jump-back)
- There are two important codepaths:
  - `TraeFlow/`: the shipping Xcode app (Bundle ID `ai.traeflow.app`)
  - `Prototype/`: a SwiftPM prototype with focused tests and reference implementations

## Start Here

- Product overview: `README.md`
- App entry: `TraeFlow/App/TraeFlowApp.swift`, `TraeFlow/App/AppDelegate.swift`
- TRAE variant model (TRAE / TRAE CN / TRAE WORK / TRAE WORK CN): `TraeFlow/Models/TraeVariant.swift`, `TraeFlow/Models/ClientProfile.swift`
- Flow Island left region (compact feature / expanded feature container / session detail): `TraeFlow/UI/Views/FlowIslandLeftRegion.swift` (expanded slot renders `LeftFeatureContainerView`), `TraeFlow/UI/Views/NotchView.swift` headerRow (compact slot)
  - Left feature system: `LeftFeatureStore` (enabled features, `compactFeatureID`, `expandedActiveFeatureID`, ordering, migration from legacy dual-selection) — `TraeFlow/Services/LeftFeatures/LeftFeatureStore.swift`
  - Built-in features: music (`NowPlayingProvider` via MediaRemote private framework, dlopen-loaded), shelf (`ShelfStore` for temporary file transfer with AirDrop), newsnow (NewsNow remote instance), and mineradio (Mineradio Bridge compat layer) — `TraeFlow/Services/LeftFeatures/Music/`, `TraeFlow/Services/LeftFeatures/Shelf/`, `TraeFlow/Services/LeftFeatures/Mineradio/`
  - Left feature kinds: built-in `music` / `shelf` / `newsnow(baseURL:)` / `mineradio(pageURL:)`, user `customArea` (local HTML directory via `CustomAreaStore`), user `webURL` (remote URL loaded directly by `CustomAreaWebView` with `ContentSource.remoteURL`). `LeftFeature.customIconName` overrides the default SF Symbol; `LeftFeature.customDisplayName` holds the `.webURL` name. When `appendWebURLFeature` is called without an explicit `iconName`, `FaviconFetcher` auto-fetches the site favicon and writes `img:favicon-<host>.png` to `customIconName`; URL edits re-fetch if the current icon is still an auto-fetched favicon. Built-in `newsnow` and `mineradio` features auto-fetch their site favicon on first launch via `LeftFeatureStore.fetchBuiltinFaviconIfNeeded` (only when `customIconName` is nil; skipped if already fetched or user-customized). The new/edit feature sheets (`addCustomAreaSheet` in `SettingsWindowView`, `EditableCustomAreaView`) place the URL field first in webURL mode; on URL change (debounced 600ms) `FaviconFetcher.fetchMetadata` fetches both favicon and site `<title>` in parallel and auto-fills the icon and name fields — name is overwritten only if empty or still the last auto-filled value; icon is overwritten only if unset or still an auto-fetched favicon; user-edited values are preserved.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ccsonicc333/trae-flow](https://github.com/ccsonicc333/trae-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
