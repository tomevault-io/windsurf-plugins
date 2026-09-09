---
trigger: always_on
description: Operating context for AI agents (opencode, Cursor, Claude Code, etc.)
---

# AGENTS.md

Operating context for AI agents (opencode, Cursor, Claude Code, etc.)
working on this repository. Read this file first; it covers what the
project is, how to build/verify it, and the conventions to follow.

## What this repo is

`GooseRelayVPN-AndroidClient` is the Android client for the
GooseRelayVPN project. It wraps the upstream Go core (in `internal/`,
shared with the server) in an Android `VpnService` and exposes a
Jetpack Compose UI for VPN lifecycle, profile management, logs, and
settings.

Upstream Go core: <https://github.com/kianmhz/GooseRelayVPN>
This client: <https://github.com/ArashAfkandeh/GooseRelayVPN-AndroidClient>

## Architecture (Android side)

```
┌─────────────────────────────────────────────┐
│   Android UI Layer (Compose)                │
│  Home │ Profiles │ Settings │ Logs │ Info   │
└─────────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────────┐
│   ViewModels & Repository Layer             │
│  Room │ DataStore │ Hilt DI                 │
└─────────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────────┐
│   VPN Service & Go Core                     │
│  tun2socks │ GooseRelay Core (Go mobile)    │
└─────────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────────┐
│   Network Layer                             │
│  SOCKS5 → Google Apps Script → VPS exit     │
└─────────────────────────────────────────────┘
```

- `android/` — Gradle project (`settings.gradle.kts`, `build.gradle.kts`, `app/`). Kotlin 2.1.0,
  AGP 8.13.0, `minSdk=21`, `targetSdk=36`, `compileSdk=36`, JVM 17.
- `android/app/src/main/java/com/gooserelay/gooserelayvpn/` — Kotlin source:
  - `service/` — `GooseRelayVpnService` (the VpnService),
    `VpnTileService` (Quick Settings tile), `BootReceiver` (boot,
    currently a no-op).
  - `ui/` — Compose screens and ViewModels, split by feature
    (`home/`, `profiles/`, `settings/`, `logs/`, `info/`,
    `navigation/`).
  - `data/local/` — Room (`AppDatabase` v4, `ProfileEntity`,
    `ProfileDao`, `ProfileMigrations`).
  - `data/repository/` — `ProfileRepository` (Hilt `@Singleton`).
  - `dns/` — Kotlin FakeDNS interceptor (separate from the Go-side
    FakeDNS in `mobile/tun/`).
  - `di/` — Hilt `AppModule`.
  - `util/` — `VpnManager` (singleton bridge between UI and Go
    core), `ConfigGenerator` (profile → JSON config),
    `GlobalSettingsStore` (DataStore-backed).
- `mobile/` — Go mobile bridge, gomobile bind target:
  - `mobile/mobile.go` — `StartClient` / `StopClient` /
    `StartTun` / `StopTun` / `StartTunBridge` / `StopTunBridge`
    exported to Kotlin via `mobile.Mobile.*`.
  - `mobile/tun/` — FakeDNS proxy (`fakedns_proxy.go`),
    DNS mapper (`dns_mapper.go`), `tun_api.go` (exported
    `StartFakeDNSProxy` etc.).
- `internal/` — upstream Go core (carrier, session, socks, config,
  etc.). **Treat as read-only when working on the Android client.**
- `apps_script/` — Google Apps Script deployment that fronts the
  carrier traffic over Google infrastructure.

## How to build

```bash
# 1. Build the Go mobile AAR (requires Go 1.25+, NDK installed).
# On Windows: use build_go_mobile.bat; on Unix: build_go_mobile.sh.
bash ./android/build_go_mobile.sh
# Output: android/app/libs/gooserelayvpn.aar

# 2. Build the debug APK.
cd android
./gradlew :app:assembleDebug
# Output: android/app/build/outputs/apk/debug/GooseRelayVPN.apk

# 3. Build the release AAB (requires signing config in local.properties).
./gradlew :app:bundleRelease
```

## How to verify

```bash
cd android

# Unit tests (JVM, no emulator needed).
./gradlew :app:testDebugUnitTest --stacktrace

# Lint.
./gradlew :app:lintDebug

# Compile-only fast check.
./gradlew :app:compileDebugKotlin --stacktrace

# Go-side vet + format.
go vet ./mobile/...
gofmt -l mobile/
```

Instrumented tests (`./gradlew :app:connectedDebugAndroidTest`)
require a connected emulator or device; CI does not run them.

## Conventions

### Kotlin

- Style: `kotlin.code.style=official` (see
  `android/gradle.properties`).
- ViewModels: `@HiltViewModel` + constructor injection. See
  `ui/profiles/ProfilesViewModel.kt` as the canonical example.
- Singletons: `object` declarations for cross-cutting state
  (`VpnManager`, `ConfigGenerator`, `GlobalSettingsStore`).
- Error handling: `runCatching { ... }.onFailure { ... }` for
  non-fatal failures; `try { ... } catch (_: Exception) {}` only
  for true background noise (e.g. closing sockets during shutdown).
- Coroutines: `CoroutineScope(SupervisorJob() + Dispatchers.X)`.
  Use `withContext(Dispatchers.IO)` for thread hops; avoid nested
  `launch` blocks inside a coroutine. Guard with `isActive` after
  suspending operations.
- Logging: `android.util.Log` for system logcat; `VpnManager.appendLog`
  for user-visible log lines (shown in the Logs screen, with a
  2000-line ring buffer). **Never log credentials** —
  `ProfileEntity.socksPass`, `tunnelKey`, `scriptKeysText` (and their
  JSON-serialized forms in `ConfigGenerator.exportProfileJson`) must
  not appear in logs.

### Go (in `mobile/`)

- `gofmt`-clean; `go vet ./mobile/...` returns no findings.
- Exported functions PascalCase (gomobile convention). Logger uses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hidden-Node/GooseRelayVPN-AndroidClient](https://github.com/Hidden-Node/GooseRelayVPN-AndroidClient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
