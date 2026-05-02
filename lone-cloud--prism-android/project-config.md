---
trigger: always_on
description: - Do NOT add obvious comments that just describe what the code does
---

# GitHub Copilot Instructions

## Code Style

- Do NOT add obvious comments that just describe what the code does
- Only add comments for complex logic, non-obvious behavior, or important context
- Prefer self-documenting code with clear variable and function names over comments
- Avoid redundant comments like "// Create button" or "// Set text color"
- Do NOT add copyright headers to new files

## Validation

After every code change, run:

```sh
./gradlew ktlintFormat compileDebugKotlin
```

Fix all errors before finishing. `ktlintFormat` auto-fixes formatting; if it reformats files, include those changes. For a full lint pass: `./gradlew ktlintCheck detekt`.

## What Prism Android Is

Prism Android is a **UnifiedPush distributor**. It maintains a persistent WebSocket to a Mozilla Autopush-compatible or self-hosted Prism server, decrypts RFC 8291 WebPush payloads, and routes push notifications to registered apps.

Two app types are supported:
- **Standard UP apps**: receive encrypted payloads forwarded via UnifiedPush IPC (`Distributor.sendMessage`)
- **Manual apps**: registered directly with the Prism server; Prism Android decrypts + displays Android notifications with rich actions

## Notification Flow

```
Push server (WebSocket)
  └─ ServerConnection.onMessage()
       └─ deserialize ServerMessage
            ├─ Notification + manual app? → decrypt (WebPushDecryptor) → ManualAppNotifications.showNotification()
            └─ Notification + UP app?     → Distributor.sendMessage() → IPC to target app
```

## Architecture

### Two-Process Split

- `:ui` process: `MainActivity` + all Compose UI
- main process: `FgService`, `ServerConnection`, `PrismServerClient`, all receivers

IPC between processes uses the UP library: `InternalMessenger` (UI side) ↔ `PrismInternalService` (main side). UI sends config changes via `PrismConfigReceiver` broadcasts; main process sends UI updates via `sendUiAction` / `subscribeUiActions`.

### WebSocket Lifecycle

```
FgService.sync()
  → ServerConnection.start()           # builds OkHttp WS request
  → MessageSender.newWs(ws)            # registers WS for outbound sends
  → SourceManager.setConnected()       # tracks state
  → RestartWorker (WorkManager)        # periodic reconnect + network-change reconnect
```

`ServerConnection.onHello()` persists the tested push URL via `ApiUrlCandidate.finish()`.

### Manual App Registration

1. UI calls `MainViewModel.addManualApp(name, targetApp?)`
2. Generates VAPID keypair (`VapidKeyGenerator`) + WebPush encryption keys (`WebPushEncryptionKeys`)
3. Stores in UP library DB with pipe-delimited description: `target:<pkg>|vp:<vapidPrivKey>|sid:<subId>`
4. Sends `ClientMessage.Register(channelID, vapidPublicKey)` to push server
5. Server responds → `ServerConnection.onRegister()` → `ManualAppRegistrationHandler.handleRegister()`
6. Handler calls `PrismServerClient.registerApp()` to register with Prism server HTTP API

### Description Field Encoding (`DescriptionParser`)

The UP library `Store.description` field encodes manual app metadata as pipe-delimited key:value pairs:

```
target:<packageName>|vp:<base64UrlVapidPrivKey>|sid:<subscriptionId>
```

- `isManualApp(description)` checks `startsWith("target:")`
- `extractValue(description, "vp:")` → VAPID private key
- `extractValue(description, "sid:")` → Prism server subscription ID

### Security

| Layer | Mechanism |
|---|---|
| API key storage | AES-256-GCM, AndroidKeyStore-backed (`SecureStringPreferences`) |
| Per-channel WebPush keys | Same — `EncryptionKeyStore` with key alias `"prism_webpush_encryption_keys"` |
| WebPush decryption | RFC 8291 `aes128gcm` — Tink ECDH + HKDF + JCE AES-GCM (`WebPushDecryptor`) |
| Notification action URLs | Same-origin validated against Prism server URL (`resolveAndValidateActionUrl`) |

Never relax the same-origin check. Never store sensitive values in plain SharedPreferences.

## Source Layout

All Kotlin under `app/src/main/java/app/lonecloud/prism/`:

```
PrismApplication.kt         # Application; manual WorkManager init
AppScope.kt                 # Global SupervisorJob + Dispatchers.IO coroutine scope
PrismPreferences.kt         # All persistent config + per-channel token storage
SecureStringPreferences.kt  # AES-256-GCM encrypted SharedPreferences (AndroidKeyStore)
PrismServerClient.kt        # HTTP client: register/delete/test subscriptions
EncryptionKeyStore.kt       # Per-channel ECDH key storage (via SecureStringPreferences)
DatabaseFactory.kt          # UP library DB singleton
PrismConfig.kt              # UP library AppConfig (restartable, no login, migration)
Distributor.kt              # UnifiedPushDistributor: register/unregister channels
MigrationManager.kt         # UP migration support

api/
  ServerConnection.kt           # WebSocket client + message dispatch
  ApiUrlCandidate.kt            # State machine for testing a new push server URL
  ManualAppRegistrationHandler.kt  # Handles Register response → PrismServerClient.registerApp
  MessageSender.kt              # Thread-safe WS send + ping rate limiter
  data/
    ClientMessage.kt            # Outbound WS messages (sealed class, @Serializable)
    ServerMessage.kt            # Inbound WS messages (sealed class, @Serializable)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lone-cloud/prism-android](https://github.com/lone-cloud/prism-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
