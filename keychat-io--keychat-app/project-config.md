---
trigger: always_on
description: **All code-related content MUST be written in English:**
---

# Keychat — AI Coding Agent Instructions

## 🌍 Language Policy (CRITICAL)

**All code-related content MUST be written in English:**

- ✅ Code comments (both `///` and `//`)
- ✅ Function/class/variable names
- ✅ Documentation strings (docstrings)
- ✅ Git commit messages
- ✅ Error messages in code
- ✅ Test descriptions
- ✅ TODO/FIXME notes

**Only user-facing content uses localized languages:**

- UI text (via i18n/localization)
- User error messages (displayed through `EasyLoading`)
- Documentation for end users

**Why:** This is an international open-source project. English ensures all developers worldwide can contribute and maintain the codebase.

## Project Overview

Keychat is a privacy-focused secure chat super-app for Bitcoiners. It's a **Flutter + Rust monorepo** managed by **Melos 7.3.0**, using **FVM** (Flutter 3.38.2, Dart ≥3.9.0).

Three packages: `packages/app` (main Flutter app), `packages/keychat_ecash` (Bitcoin ecash wallet UI/logic), `packages/keychat_rust_ffi_plugin` (Rust FFI for crypto — Signal, MLS, Nostr, Cashu).

**Design Philosophy:** "Postal system" metaphor — users send messages "stamped" with Bitcoin ecash to Nostr relays (post offices), which collect ecash and deliver encrypted messages. No registration required.

## Architecture

**State management:** GetX exclusively — `Get.put`/`Get.lazyPut` for DI, `.obs` reactive variables, `Obx()` in UI, `Get.find<T>()` for lookup.

**Database:** Isar Community 3.3.0 (NoSQL) for app data; SQLite (via Rust) for Signal/MLS/Cashu state. `SharedPreferences` via `Storage` for settings; `FlutterSecureStorage` via `SecureStorage` for keys/mnemonics.

**Encryption strategy pattern:** `BaseChatService` abstract class → `SignalChatService` (1:1), `MlsGroupService` (groups), `Nip4ChatService` (legacy). Dispatch based on `Room.encryptMode`.

**Nostr message flow:** `WebsocketService` → multiple `RelayWebsocket` instances → events queued in `NostrAPI.nostrEventQueue` (async sequential) → routed to appropriate chat service.

**Rust FFI:** `flutter_rust_bridge` v2.11.1 generates Dart bindings. Each Rust module (`api_signal.rs`, `api_mls.rs`, `api_nostr.rs`, `api_cashu.rs`) has a `lazy_static` Mutex-guarded store with dedicated Tokio runtime. Import with aliases:

```dart
import 'package:keychat_rust_ffi_plugin/api_nostr.dart' as rust_nostr;
import 'package:keychat_rust_ffi_plugin/api_signal.dart' as rust_signal;
```

### NIP-17/44/59 Implementation (Critical)

**Message encryption layers** (innermost to outermost):

1. **Rumor** (kind 14) — actual message content + metadata
2. **Seal** (kind 13) — encrypted rumor with NIP-44, reveals sender to recipient
3. **Gift Wrap** (kind 1059) — encrypted seal with random key, timestamp tweaked ±2 days

**Two signing paths:**

- **Amber Signer** (Android external signer): Uses `SignerService.getNip59EventString()` — single signature via Amber API
- **Rust FFI** (native keys): Uses `rust_nostr.createGiftJson()` — all encryption/signing in Rust

**Sender Copy architecture** (currently disabled, reserved for future multi-device sync):

- `SignerService.getNip59EventStringsWithSenderCopy()` creates TWO gift wraps per message
- One encrypted for receiver (DH with receiver's pubkey)
- One encrypted for sender (DH with sender's own pubkey, enables multi-device retrieval)
- `sendAndSaveNostrEventWithSenderCopy()` handles dual-event sending
- When enabled: Amber signs twice, both events sent to relays with same rumor ID for deduplication

## Key Conventions

### Service singletons — always use this pattern:

```dart
class FooService extends BaseChatService {
  FooService._();
  static FooService? _instance;
  static FooService get instance => _instance ??= FooService._();
}
```

### File naming:

| Type        | Pattern               | Example                          |
| ----------- | --------------------- | -------------------------------- |
| Service     | `xxx.service.dart`    | `room.service.dart`              |
| Controller  | `xxx.controller.dart` | `chat.controller.dart`           |
| Page/Screen | `xxx_page.dart`       | `chat_setting_contact_page.dart` |
| Generated   | `xxx.g.dart`          | `room.g.dart`                    |

### Isar models use `@Collection` with code generation (`part '*.g.dart'`). Enums use `@Enumerated(EnumType.ordinal32)`. Embedded objects live in `models/embedded/`.

### Error handling pattern:

```dart
try {
  // logic
} catch (e, s) {
  logger.e('Description', error: e, stackTrace: s);
  EasyLoading.showError(Utils.getErrorMessage(e));
}
```

**Why this pattern:** Centralized error logging with stack traces + user-facing toast notifications. `Utils.getErrorMessage()` extracts localized error messages from exceptions.

### Import order: Dart core → Flutter/framework → third-party → project (`package:keychat/`) → Rust FFI (aliased).

### Comments & Documentation

**MANDATORY: All comments and documentation MUST be in English.**

- Use `///` for public API documentation (classes, methods, functions)
- Use `//` for inline implementation comments
- **Always add doc comments** when creating new functions or modifying existing ones
- Explain the "why" not just the "what" in comments

**Examples:**

```dart
// ✅ CORRECT - English comments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keychat-io/keychat-app](https://github.com/keychat-io/keychat-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
