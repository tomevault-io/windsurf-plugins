---
trigger: always_on
description: > **Developer reference** for AI coding agents and contributors. This file documents architecture decisions, implementation details, conventions, and patterns used throughout the codebase. For a user-facing overview, see [README.md](README.md).
---

# AGENTS.md — CrossX (SendToX4)

> **Developer reference** for AI coding agents and contributors. This file documents architecture decisions, implementation details, conventions, and patterns used throughout the codebase. For a user-facing overview, see [README.md](README.md).

## Project Overview

CrossX (bundle name: SendToX4) is a **native multiplatform SwiftUI app** for iOS 26+ and macOS 26+ that converts web pages to EPUB 2.0 format and sends them to an Xtreink X4 e-reader over its local WiFi hotspot. The app uses SwiftData for persistence, targets a Liquid Glass UI, and ships with an iOS Share Extension for converting pages directly from Safari.

**Bundle ID**: `com.crossappjtv.point`
**Display Name**: CrossX

## Architecture

```
SendToX4/
  Models/          — SwiftData @Model classes (Article, DeviceSettings, ActivityEvent, QueueItem)
  Views/           — SwiftUI views (Liquid Glass, iOS 26 / macOS 26 adaptive)
  ViewModels/      — @Observable view models with async orchestration (@MainActor)
  Services/        — Business logic (EPUB, device communication, content extraction)
  Intents/         — App Intents for Siri Shortcuts (ConvertURLIntent, CrossXShortcuts)
  Utilities/       — Pure helpers (HTML sanitization, string extensions, design tokens)
  Resources/       — Bundled assets (readability.js)

SendToX4ShareExtension/  — iOS-only Share Extension (receives URLs from Safari)
```

### Data flow

```
Views → ViewModels → Services → Device (HTTP) / SwiftData (persistence)
```

- **Views** observe `@Observable` ViewModels and call methods on user interaction
- **ViewModels** are `@MainActor`, orchestrate service calls, and manage UI state
- **Services** are `nonisolated`, perform async I/O, and throw errors on failure
- **Models** are SwiftData `@Model` classes persisted via `ModelContainer`

## Features Implemented

### Core

1. **URL-to-EPUB conversion** — full pipeline: fetch HTML → extract content → sanitize → build EPUB 2.0 → send to device
2. **Dual firmware support** — Stock firmware (`192.168.3.3`) and CrossPoint firmware (`192.168.4.1` / `crosspoint.local`)
3. **Auto-detection** — probes both firmware endpoints concurrently; CrossPoint tries mDNS first, then static IP fallback
4. **File manager** — browse, upload (`.epub`/`.xtc`/`.bump`/`.txt`), create folders, delete, move (CrossPoint only)
5. **Unified activity history** — merged timeline of EPUB conversions (`Article`) and file operations (`ActivityEvent`)
6. **iOS Share Extension** — convert and send from Safari or any app that shares URLs
7. **Native multiplatform** — iOS, iPadOS, and macOS from a single codebase (not Mac Catalyst)
8. **EPUB send queue** — persists EPUBs to disk when offline, batch-sends when device connects, with queue UI in Convert tab
9. **Siri Shortcuts** — App Intent (`ConvertURLIntent`) for URL-to-EPUB conversion from Shortcuts app, Siri, or Share Sheet automations
10. **Recent conversions** — last 3 completed conversions shown in Convert tab with inline action menus (Resend, Reconvert & Share, Copy URL)

### Content Extraction

11. **SwiftSoup heuristic extraction** — primary extractor using CSS selectors for article body, title, author, description
12. **Readability.js fallback** — WKWebView-based extraction when SwiftSoup yields < 400 characters
13. **Twitter/X extractor** — dedicated handler using fxtwitter API for tweet content and metadata
14. **Chapter splitting** — auto-splits long content at `<h2>` headings or every 50 paragraphs (15,000-char threshold)

### UI/UX

15. **Liquid Glass design** — `.glassEffect()` modifiers on iOS 26 / macOS 26
16. **Design system** — `AppColor` enum (accent/success/error/warning) with `AccentColor` asset (teal, light+dark)
17. **Platform-adaptive status bar** — iOS tab bar bottom accessory vs macOS Xcode-style bottom status bar
18. **Device status display** — firmware version, IP, WiFi mode, RSSI, free heap, uptime
19. **Expandable history rows** — tap to reveal full URL, error details, metadata
20. **History filtering** — All / Conversions / File Activity / Queue tabs with full-text search
21. **Queue section in Convert tab** — always-visible section with populated/empty glass-card states, Send All button, individual remove
22. **Queue connection prompt** — alert when device connects offering to send all queued items
23. **Settings Siri Shortcut guide** — inline 5-step setup guide with "Open Shortcuts App" button (iOS)
24. **Settings storage info** — queue EPUB count and size display with Clear Queue option
25. **Device accessory queue count** — shows "N EPUBs queued" in warning color when disconnected

## Key Design Decisions

### `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`

The project uses this build setting, making **every type implicitly `@MainActor`** unless explicitly opted out. This is the modern Swift concurrency approach but has a critical implication:

- **All service types MUST be marked `nonisolated`** — without this, service structs/protocols inherit `@MainActor`, causing stack overflows (`EXC_BAD_ACCESS code=2`) when called from async contexts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jtvargas/crosspoint-app](https://github.com/jtvargas/crosspoint-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
