---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Install dependencies
flutter pub get

# Generate localization files (required before build/run)
flutter gen-l10n

# Run on connected device
flutter run

# Analyze code for errors
flutter analyze

# Run all tests
flutter test

# Run single test file
flutter test test/encryption_test.dart

# Run tests with coverage
flutter test --coverage

# Build APK (Android)
flutter build apk

# Build iOS (requires macOS + Xcode)
flutter build ios
```

## Architecture

Flutter app for distributed file storage across NFC tags using the NFAR (NFC Archive) binary format.

### Data Flow

**Archive:** File → (compress) → (encrypt) → chunk into NFAR packets → write to NFC tags

**Restore:** Scan NFC tags in any order → collect chunks by Archive ID (UUID) → assemble → (decrypt) → (decompress) → File

### Core Layer (`lib/core/`)

- **`constants/nfar_format.dart`** — NFAR v1 binary format (28-byte header + payload + CRC32). All multi-byte values big-endian. `NfarFlags` for compression/encryption bits, `NfcTagType` enum for tag capacity calculations.
- **`models/chunk.dart`** — `Chunk` class with `toBytes()`/`fromBytes()` serialization
- **`services/chunker_service.dart`** — Splits data into chunks via `createChunks()` or `createChunksWithSize()`, reassembles with `assembleChunks()` including CRC32 validation
- **`services/encryption_service.dart`** — AES-256-GCM encryption with PBKDF2 (100k iterations). Format: salt(16) + IV(12) + ciphertext + tag(16). Use `encryptionOverhead` constant when calculating sizes.
- **`services/compression_service.dart`** — GZIP compression wrapper

### Features Layer (`lib/features/`)

Each feature follows: `data/` (repository) → `presentation/providers/` (Riverpod StateNotifier) → `presentation/screens/`

- **`nfc/`** — NFC abstraction over `nfc_manager`. `NfcRepository` manages sessions with write cooldown to prevent re-read. `NdefFormatter` converts Chunk↔NDEF with MIME type `application/vnd.nfcarchiver.chunk`.
- **`archive/`** — `ArchiveNotifier` uses sealed class states (`ArchiveInitial` → `ArchiveFileSelected` → `ArchiveConfiguring` → `ArchivePreparing` → `ArchiveReady` → `ArchiveWriting` → `ArchiveComplete`). Supports `rechunkForDetectedCapacity()` when tag is smaller than expected.
- **`restore/`** — `RestoreNotifier` with states for scanning, collecting chunks into `RestoreSession` by UUID, handling CRC errors with rescan capability.

### State Management

Riverpod with `StateNotifier` pattern using sealed classes for type-safe state transitions:
- `archiveProvider` — Archive creation workflow
- `restoreProvider` — Restore/scanning workflow

### NFAR Format

28-byte header. Flags byte: bit 0 = GZIP, bit 1 = AES-256-GCM. Archive ID is UUID v4 (16 bytes) for grouping chunks. Max 65535 chunks per archive. Chunks validated with CRC32 and can be scanned in any order.

### File Sharing (`share_plus`)

All `Share.shareXFiles` calls include explicit MIME types resolved via the `mime` package (`lookupMimeType()`) from file extensions. This is required for Telegram and other strict Android apps that validate content before enabling the send button. Without MIME types, Android's `ContentResolver` reports `application/octet-stream` and the receiving app may refuse to send.

`AndroidManifest.xml` declares `SEND` and `SEND_MULTIPLE` intent queries for proper share target resolution on Android 11+ (API 30+ package visibility).

### Version Display

Version and build number are read at runtime via `PackageInfo.fromPlatform()` (`package_info_plus` package) — **never hardcoded**. `pubspec.yaml` `version:` field is the single source of truth. The version propagates to:
- **Home screen footer**: `"NFC Archiver v1.0.10 (Build 10) © 2026"` via parameterized `versionFooter` l10n key
- **About dialog**: `applicationVersion` parameter in `showAboutDialog()`
- **Android APK**: `build.gradle` reads `flutter.versionName`/`flutter.versionCode` from pubspec

To release a new version: bump `version: X.Y.Z+N` in `pubspec.yaml` (increment both version name and build number). Nothing else needs updating.

### Localization

Uses Flutter's `gen-l10n` with ARB files in `lib/l10n/`. Supported: English (`app_en.arb`), Russian (`app_ru.arb`), Turkish (`app_tr.arb`), Ukrainian (`app_uk.arb`), Georgian (`app_ka.arb`), Polish (`app_pl.arb`), Belarusian (`app_be.arb`). Run `flutter gen-l10n` after modifying ARB files. All new UI strings must be added to `app_en.arb` (template) and all 6 translation files.

## Apple App Store Publishing

**Goal:** Publish NFC Archiver to the Apple App Store.

### Steps to Resolve

1. **Apple Developer Account** — Enroll in the Apple Developer Program ($99/year) if not already enrolled
2. **App Store Connect setup** — Create the app record in App Store Connect with bundle ID, app name, and category
3. **App icons & screenshots** — Prepare required app icon sizes (1024x1024 for store) and screenshots for all required device sizes (6.7", 6.5", 5.5" iPhones; iPad Pro)
4. **App Store metadata** — Write app description, keywords, subtitle, promotional text, and select appropriate categories

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mezinster/nfcarchiver](https://github.com/mezinster/nfcarchiver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
