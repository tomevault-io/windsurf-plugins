---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

darkVault is an Android app (Kotlin + Jetpack Compose) that provides zero-knowledge encrypted file storage backed by Google Drive. Files are encrypted on-device with AES-256-GCM before upload; Google never sees plaintext.

## Commands

```bash
# Build
./gradlew assembleDebug
./gradlew assembleRelease

# Unit tests (JVM only, no device needed)
./gradlew test

# Single test class
./gradlew test --tests "com.darkvault.app.KeyZeroingTest"

# Lint
./gradlew lint

# Instrumented tests (requires connected device/emulator)
./gradlew connectedAndroidTest
```

## Architecture

Single-activity app. `MainActivity` hosts `DarkVaultNavGraph`, which is driven entirely by `AuthState` from `AuthViewModel`. Navigation is reactive: when `authState` changes, the nav graph pops to the matching route.

**Auth state machine** (`AuthViewModel`):
```
Init → SignIn → CheckingVault → Setup (new user) → Home
                              → Unlock (returning user) → Home
                AppLocked (biometric gate; DEK still in memory) → Home
                NeedsConsent (Drive scope not granted) → ...
```

**In-memory session** (`VaultSession` singleton):
- Holds the DEK (`dek: ByteArray?`), master password, and Google account after unlock
- Zeroed on lock/sign-out via `clearDek()` — keys are never persisted to disk
- `UploadForegroundService` reads credentials from here (no disk storage)

**Encryption layers** (read before touching crypto code):

1. **DEK** (Data Encryption Key) — a random 256-bit AES key, generated once at setup
2. **KEK** (Key Encryption Key) — derived from the master password via PBKDF2-SHA256 (100k iterations), used only to wrap/unwrap the DEK
3. **vault.key** on Drive — JSON file (`VaultKeyBundle`) holding the DEK wrapped twice: once with the KEK (password path) and once with the recovery key. See `VaultKeyManager` for the 60-byte wrapped blob format.
4. **Vault files** — each file is gzip-compressed then AES-256-GCM encrypted using the DEK (`VERSION_DEK_GCM = 0x03`). A version byte is written as the first byte; `CryptoManager.decrypt()` dispatches on it and handles two legacy formats.

**File format versions** in `CryptoManager`:
- `0x03` (current): DEK-based, GZIP + AES-GCM — use `encryptWithDek()` for all new uploads
- `0x02` (legacy): per-file key derivation from password, GZIP + AES-GCM
- `0x00`-`0x01` (legacy): no version byte, salt is the raw first bytes

**Key packages:**
- `crypto/` — `CryptoManager` (file encrypt/decrypt), `VaultKeyManager` (DEK wrap/unwrap), `BiometricKeyManager` (Android Keystore key for biometric), `BiometricHelper`
- `drive/` — `DriveApiClient`: raw OkHttp calls against Drive REST API v3 with exponential-backoff retry. `getStorageQuotaOnly(knownVaultBytes)` avoids a redundant `listItems` call on the home screen.
- `data/` — `PreferencesManager`: DataStore-backed persistence for auth state, biometric IV/ciphertext, vault folder ID, settings, cache cap, and the offline vault key cache (`cached_kek_salt` / `cached_wrapped_dek`). See `cacheVaultKeyLocally()` / `getCachedVaultKey()`.
- `cache/` — three-tier cache layer:
  - `EncryptedFileCache` — session-scoped in-memory LRU (64 MB) of encrypted `ByteArray` values, keyed by `(fileId, modifiedTime)`. Cleared when `VaultSession.clearDek()` is called.
  - `LocalVaultCache` — persistent encrypted-file disk cache (`filesDir/vault_cache/`). Index is a plain JSON file (no filenames — only opaque IDs, sizes, timestamps). Per-file encrypted metadata sidecar. LRU eviction by `lastAccessedMs`, respecting `isPinned`. `evict(fileId)` removes a single entry (called on permanent delete). Upload-staged entries initially have `modifiedTime=""` and adopt the real Drive timestamp on first access rather than being evicted.
  - `FolderMetadataStore` — encrypted folder listing cache (`filesDir/folder_meta/`). Enables stale-while-revalidate on cold start: decrypted listing emitted instantly before Drive refresh. `allCachedFiles(dek)` aggregates every cached folder listing — used to build the offline file index.
- `service/` — `UploadForegroundService` + `UploadState` + `ReadyToUploadJob`: two-stage parallel pipeline. **N=3 encryption workers** (`Dispatchers.Default`) write to `cacheDir/encrypt_staging/<jobId>.vault`; **M=2 upload workers** (`Dispatchers.IO`) consume `ReadyToUploadJob` from a `Channel`. Thumbnail companions (scaled JPEG, re-encrypted with DEK) are generated and uploaded first so their Drive ID can be embedded in the main file's `appProperties.thumbnailId`.
- `viewmodel/` — `AuthViewModel` (auth state machine + lock/session timers + `isOffline` StateFlow), `HomeViewModel` (file listing, download, delete, trash, `clearLocalCache()`, `setOfflinePinned()`). `HomeViewModel` exposes `isOffline` and `offlineFiles` StateFlows; `refreshOfflineFiles()` rebuilds `offlineFiles` from `LocalVaultCache.pinnedFileIds()` + `FolderMetadataStore.allCachedFiles()`. `resetInMemoryState()` clears in-memory state only (called on `CheckingVault`); `clearDriveState()` additionally wipes disk caches (called only on sign-out / account switch).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scap3sh4rk/darkVault](https://github.com/scap3sh4rk/darkVault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
