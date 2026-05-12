---
trigger: always_on
description: Guidance for Claude Code working in this repo. Keep this file lean — only record gotchas and conventions that aren't derivable from reading the code.
---

# CLAUDE.md

Guidance for Claude Code working in this repo. Keep this file lean — only record gotchas and conventions that aren't derivable from reading the code.

## Project

YueLink (悦通) — Flutter UI + mihomo (Clash.Meta) Go core. Targets: Android, iOS, macOS, Windows, Linux. Package/Bundle ID: `com.yueto.yuelink`. iOS App Group: `group.com.yueto.yuelink`.

## Build

```bash
flutter pub get
dart setup.dart build -p <android|ios|macos|windows> [-a <arch>]
dart setup.dart install -p <platform>          # copies libs into platform dirs (gitignored)
dart setup.dart clean
flutter run                                    # mock mode if no native lib
flutter analyze --no-fatal-infos --no-fatal-warnings
flutter test
flutter build apk|ios|macos|windows
```

Requires Go ≥ 1.22, Flutter ≥ 3.38.4, Dart ≥ 3.10.3. CI uses Flutter 3.41.7 / Go 1.23. Xcode ≥ 15. Android NDK r26+.

**macOS universal**: build `arm64` and `x86_64` separately, then `install` merges via `lipo`.

**Android signed release**:
```bash
KEYSTORE_PATH="$(pwd)/android/app/yuelink.jks" KEYSTORE_PASSWORD=yuelink2024 \
KEY_ALIAS=yuelink KEY_PASSWORD=yuelink2024 \
flutter build apk --release --split-per-abi
```

Native libs install to: `android/app/src/main/jniLibs/<abi>/libclash.so`, `ios/Frameworks/libclash.a(+.h)`, `macos/Frameworks/libclash.dylib`, `windows/libs/<arch>/libclash.dll`.

## Architecture overview

```
Flutter (Dart, Riverpod) → CoreController (FFI) → hub.go (CGO) → mihomo
                         → MihomoApi (REST :9090) ← ─ ─ ─ ─ ─ ─ ─ ─┘
                                                                    ↕
                                                   Platform VPN (TUN / system proxy)
XBoardApi (HTTPS) → CloudFront → XBoard panel
```

Unified `ClashCore` interface (`lib/core/clash_core.dart`): `RealClashCore` dispatches lifecycle to FFI and data to REST; `MockClashCore` routes to `CoreMock`. Callers always go through `CoreManager.instance.core.X()`. Streaming (traffic / connections / logs websockets) lives in dedicated repositories — real-mode only; mocks are timer-based polls.

`lib/` is flat — no legacy `pages/` `services/` `providers/` `ffi/` `l10n/`. Top-level: `core/`, `domain/`, `infrastructure/`, `modules/`, `shared/`, `i18n/`, `theme.dart`, `constants.dart`, `main.dart`. New features go in `lib/modules/<feature>/`.

`core/managers/` — one file per external system: `core_lifecycle_manager`, `core_heartbeat_manager`, `system_proxy_manager`. `core/kernel/` holds `core_manager.dart` (8-step startup), `config_template.dart`, `geodata_service.dart`, `overwrite_service.dart`, `process_manager.dart`, `recovery_manager.dart`. `core/providers/core_provider.dart` is thin Riverpod wiring (~160 lines).

`infrastructure/datasources/xboard/` is a 5-file module (`client`, `errors`, `models`, `api`, `index` barrel). Always import via `index.dart`.

`appVersionProvider` reads version from `pubspec.yaml` at runtime — no hardcoded version. CI passes `--build-name` with `-pre` suffix for pre-releases.

## Platform VPN

| Platform | Mechanism |
|----------|-----------|
| Android | `VpnService` + TUN fd (always, regardless of connectionMode) — `android/.../YueLinkVpnService.kt` |
| iOS / TrollStore | `NEPacketTunnelProvider` separate process; AppDelegate handles `startVpn`/`stopVpn`/`resetVpnProfile`/`clearAppGroupConfig`; 20s timeout; `backgroundVpnObserver` sends `vpnRevoked`. Min iOS 15. |
| macOS | `networksetup` on ALL interfaces (`_verifySystemProxy` checks each) |
| Windows | Registry |

MethodChannel name: `com.yueto.yuelink/vpn`.

## Critical conventions (gotchas — not derivable from code)

### Core / FFI
- **Default connection mode is `systemProxy`** (not TUN). Mobile always uses VPN regardless; setting only applies to desktop.
- iOS Go core must be `c-archive`, not `c-shared`. Extension memory budget is ~50 MB on iOS 15+ (Apple raised it from the iOS 14 ~15 MB cap; confirmed via Apple Dev Forums #106377). Code in this repo still treats the budget as scarce — drop Dart-heap config strings after Swift writes them to App Group, keep `mtu: 1500`, keep `geodata-loader: memconservative`. The extra headroom is for absorbing large subscriptions, not a license to oversize buffers.
- All Go state behind a single mutex (`state.go`).
- **All Go exports return `*C.char`**: empty = success, non-empty = error, NULL pointer = success (Go can return NULL after panic recovery). `CoreController._callStringFn` handles all three.
- **Never use `Isolate.run()` for FFI**. Re-opening `DynamicLibrary` in a new isolate hangs on Android/macOS. `InitCore` (~1s) and `StartCore` (~2s) run synchronously on main isolate. Same for pure Dart config processing — <10ms, no isolate needed.
- `CoreManager` handles VPN per-platform — `CoreActions` must NOT call `VpnService` directly.
- FFI bindings (`core_bindings.dart`) cover lifecycle + MITM control only. Data goes through `MihomoApi`. All failable exports return `Pointer<Utf8>`.

### Android
- VPN permission always requested (no connectionMode guard).
- `POST_NOTIFICATIONS` requested at runtime in `MainActivity.onStart()` for API 33+. Without it the foreground notification is suppressed and the service may be killed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onesyue/yuelink](https://github.com/onesyue/yuelink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
