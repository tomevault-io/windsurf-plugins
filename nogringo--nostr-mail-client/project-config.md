---
trigger: always_on
description: This file contains project-specific context for AI coding agents. Read it before making changes.
---

# AGENTS.md - Nostr Mail Client (Nmail)

This file contains project-specific context for AI coding agents. Read it before making changes.

---

## Project Overview

**Nostr Mail Client** (branded as **Nmail**) is a cross-platform Flutter email client built on the [Nostr protocol](https://github.com/nostr-protocol/nips). Users own their identity (`npub`/pubkey) and local data; messages are transported through Nostr relays instead of a central mail provider.

The repository is now a Dart/Flutter workspace with a shared core package and two app distributions:

- `packages/nmail_core` - shared Nmail app, UI, routing, services, models, localization, storage, and Nostr/mail logic.
- `apps/nmail_standard` - standard distribution with Firebase/FCM push support.
- `apps/nmail_foss` - FOSS distribution with UnifiedPush support and no Firebase dependency.

Current package metadata:

- Workspace SDK constraint: `^3.12.2`
- App version: `0.14.2+26` in both app pubspecs
- Core package version: `0.0.1`
- Primary platforms: Android, Linux, Web
- Also present: iOS, macOS, Windows runner folders

---

## Technology Stack

| Layer | Technology |
|-------|------------|
| Framework | Flutter |
| Language | Dart |
| Workspace layout | Dart pub workspace (`pubspec.yaml` with `apps/*` and `packages/*`) |
| State management / DI | `get` (GetX) |
| Routing | `go_router` via `MaterialApp.router` |
| Nostr protocol | `ndk` + `ndk_flutter` |
| Email domain logic | `nostr_mail` |
| Address book | `nostr_address_book`, `vcard_dart` |
| Local database / cache | `sembast`, `sembast_web`, `sqflite`, `sqflite_common_ffi`, `idb_shim` |
| Offline queues | `broadcast_queue_shim_for_ndk`, `blossom_upload_queue_shim_for_ndk` |
| Attachments / Blossom | `blossom_cache`, `file_picker`, `file_saver` |
| Rich text editor | `flutter_quill`, `markdown_quill`, `vsc_quill_delta_to_html` |
| MIME mail construction | `enough_mail_plus` |
| HTML/PDF rendering | `flutter_widget_from_html_core`, `pdfrx` |
| Push notifications | FCM in `nmail_standard`; UnifiedPush in `nmail_foss`; shared registration logic in core |
| Local notifications | `flutter_local_notifications` |
| Toast notifications | `toastification` |
| Theming | `system_theme` + custom persisted color schemes |
| Desktop window chrome | `window_manager` |
| Localization | Flutter gen-l10n from `packages/nmail_core/lib/l10n/*.arb` |

---

## Repository Structure

```text
.
|-- pubspec.yaml                         # Workspace root (`apps/*`, `packages/*`)
|-- pubspec_overrides.yaml               # Optional local dependency overrides
|-- apps/
|   |-- nmail_standard/
|   |   |-- lib/main.dart                # Calls runNmailApp(onReady: FcmPush.init)
|   |   |-- lib/push/fcm_push.dart       # Firebase Messaging integration
|   |   |-- lib/firebase_options.dart    # Injected/validated by CI for release builds
|   |   `-- firebase.json
|   `-- nmail_foss/
|       |-- lib/main.dart                # Calls runNmailApp(onReady: UnifiedPushHandler.init)
|       `-- lib/push/unified_push.dart   # UnifiedPush foreground/background entry points
|-- packages/
|   `-- nmail_core/
|       |-- lib/app/bootstrap.dart       # Shared initialization and MainApp
|       |-- lib/app/routes/              # go_router route tree and route constants
|       |-- lib/app/config/              # App/distribution config
|       |-- lib/config/nostr_config.dart # Bootstrap relays and recommended defaults
|       |-- lib/controllers/             # GetX controllers
|       |-- lib/models/                  # Plain Dart models
|       |-- lib/services/                # Long-lived services and platform abstractions
|       |-- lib/utils/                   # Pure helpers/extensions
|       |-- lib/views/                   # Feature screens and shared shells/layouts
|       |-- lib/widgets/                 # Reusable widgets
|       |-- lib/l10n/                    # ARB files + generated localizations
|       `-- test/                        # Unit tests
|-- .github/workflows/                   # CI/CD for web, Android, Linux, macOS, releases
`-- scripts/check_android_16kb_alignment.sh
```

Approximate Dart file counts:

- `packages/nmail_core/lib`: about 207 Dart files
- app distribution `lib` folders: 5 Dart files total
- tests: 10 `*_test.dart` files in `packages/nmail_core/test`

---

## Build, Run, And Test Commands

Run commands from the directory shown in the command when possible. The CI workflows mostly run `flutter pub get` inside each app directory.

```bash
# Standard app dependencies
cd apps/nmail_standard
flutter pub get

# FOSS app dependencies
cd apps/nmail_foss
flutter pub get

# Core package tests
cd packages/nmail_core
flutter test

# Analyze core
cd packages/nmail_core
flutter analyze

# Analyze an app distribution
cd apps/nmail_standard
flutter analyze

# Run standard distribution
cd apps/nmail_standard
flutter run -d chrome
flutter run -d linux
flutter run -d android

# Run FOSS distribution
cd apps/nmail_foss
flutter run -d linux
flutter run -d android

# Build web (standard distribution)
cd apps/nmail_standard
flutter build web --release --output ../../build/web

# Build Android standard distribution
cd apps/nmail_standard
flutter build apk --release

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nogringo/nostr-mail-client](https://github.com/nogringo/nostr-mail-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
