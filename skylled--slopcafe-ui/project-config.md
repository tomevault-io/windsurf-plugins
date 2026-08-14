---
trigger: always_on
description: handles it here. `ReadTextResponse` (`GET /d/{id}/text`) is likewise generated
---

# ♊ GEMINI.md — Repository Grounding & Codebase Map

Welcome! This file serves as a grounding and reference document for future AI developer sessions. It outlines the purpose of the Slopcafe Operator App (UI), maps key files and features to their respective implementations, and establishes the "self-healing" documentation protocol.

---

## 📌 Repository Purpose
The **Slopcafe Operator App** is a professional, feature-rich Flutter administrative application designed for operators to manage, search, monitor, and configure the Slopcafe fleet deployment.

It communicates with the Slopcafe Backend API to perform fleet management tasks, agent registrations, document revocation, and metadata indexing.

---

## 🛠️ Technology Stack
- **Framework**: Flutter SDK (>=3.0.0)
- **State Management**: `flutter_riverpod` (Riverpod 3.x). The mutable stores
  (`connectionStateProvider`, `documentsListProvider`, `agentsListProvider`) use
  the modern `Notifier`/`NotifierProvider` API — each overrides `build()` for its
  initial state and reads `ref` directly (Riverpod 3 dropped the legacy
  `StateNotifier`/`StateNotifierProvider` from the default barrel).
- **HTTP Client**: `dio` (with custom interceptors for auth and status monitoring)
- **Local Persistence**: 
  - `flutter_secure_storage` (v10; secure storage of base URLs and API operator
    tokens). v10 dropped the Jetpack-Security/`EncryptedSharedPreferences` Android
    backend (deprecated by Google) for default custom ciphers; values written by
    earlier builds auto-migrate on first access, so no `AndroidOptions` are set.
  - Custom SQLite or local file-based database for offline document caching
- **Localization**: `flutter_localizations` + `intl` via Flutter's `gen-l10n` ARB
  pipeline. Every user-facing string is centralized in `lib/l10n/app_en.arb`
  (see the **Localization (i18n)** section below).
- **API Models & Error Codes**: **generated** from the backend's canonical
  **OpenAPI 3.1** contract — `freezed` + `json_serializable` data classes plus an
  `ErrorCode` enum — by a bespoke pure-Dart emitter (`tool/generate_api.dart`)
  feeding the existing `build_runner` pipeline. The hand-written `lib/models/`
  classes are gone; see **API layer (generated from the OpenAPI contract)** below.
- **URL Launching**: `url_launcher` for external browser navigation on mobile platforms (Android/iOS).
- **Inbound web links**: `app_links` — Android App Links, so a tap on a
  `https://slopcafe.com/d/:id` or `/s/:slug` URL anywhere on the device opens
  the Reader instead of a browser. **Mobile only by design** (see **Inbound web
  links** below and [docs/deep-links.md](file:///Users/kyle/Repos/slopcafe_ui/docs/deep-links.md)).
- **Platform Targets**: macOS, iOS, Android, and Web

---

## 🎨 Design Language — "Cortado"
The mobile UI follows the **Cortado** design language (a warm café metaphor): a terracotta **clay** +
**honey** OKLCH-derived palette, serif display type, text-forward "plate" cards (no cover art), tinted
tag chips, press-cards, and a floating pill tab bar. The information architecture is three tabs —
**Library** ("The Café"), **Search**, and **Operate** ("The Pass") — plus pushed routes: a full-screen
document **Reader**, a **Collections** tag browser, a generic **document list** (a tag's collection /
"see all"), and **Settings**. Copy uses "light café flavor": café-flavored section/screen titles,
professional functional copy. Both **Cortado-light** and a derived **Cortado-dark** are supported via
`ThemeMode.system`.

### Large-screen adaptation (tablet / desktop)
Cortado is phone-first but **adaptive**, via one small system in
[lib/core/design/layout.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/design/layout.dart):
* **One breakpoint** — at window width ≥ `AppLayout.railBreakpoint` (600, M3's compact/medium
  boundary) the shell swaps the floating pill tab bar for a left **side rail** (same three tabs +
  desktop affordances: an explicit **refresh** action — pull-to-refresh has no mouse gesture — and a
  Settings shortcut). Below it, the phone layout is byte-for-byte unchanged.
* **Readable columns** — every scrolling screen centers its content by growing its horizontal
  gutter (`AppLayout.gutterFor` / the `AdaptiveGutter` builder): `contentMax` 760 for
  lists/stats/compose, `formMax` 640 for Settings, `readerMax` 860 for the Reader surface. On
  phones the gutter collapses to the original `AppSpacing.screenH`, so compact layouts are
  untouched. The three shell tabs also swap `AppSpacing.bottomInset` for a small
  `context.shellBottomInset` when the floating bar is gone.
* **Wide-only layout shifts** — Operate's stat grid goes 2×2 → 4-across when the content column
  fits it; modal sheets cap at `sheetMax` 560 (centered); toasts keep a compact fixed width.
* **Pointer affordances** — `PressCard` shows a click cursor + a whisper of hover lift; plain
  tap targets use the `Tappable` wrapper (a cursor-aware `GestureDetector` drop-in, in
  `press_card.dart`).
* **macOS window** — `MainFlutterWindow.swift` sets a 400×600 `minSize` so the layout can't
  collapse below its compact floor.

---

## 🌐 Localization (i18n)
The app is wired for multiple languages using Flutter's **idiomatic `gen-l10n` ARB pipeline**, even

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Skylled/slopcafe_ui](https://github.com/Skylled/slopcafe_ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
