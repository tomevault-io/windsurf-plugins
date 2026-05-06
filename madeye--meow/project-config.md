---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Prerequisites (one-time)
cd flutter_module && flutter pub get && cd ..

# Build debug APK (arm64 only, release Rust for smaller .so)
export JAVA_HOME=/path/to/jdk17
./gradlew :mobile:assembleDebug -PTARGET_ABI=arm64 -PCARGO_PROFILE=release

# Build all ABIs
./gradlew :mobile:assembleDebug -PCARGO_PROFILE=release

# Build Rust only (faster iteration on native code)
./gradlew :core:cargoBuildArm64 -PCARGO_PROFILE=release

# Clean (includes cargo clean)
./gradlew clean

# E2E test (requires ssserver, Android emulator, adb)
# Configurable via: EMULATOR, ADB, AVD, APK, SSSERVER, SKIP_EMULATOR_BOOT
./test-e2e.sh

# Run with existing emulator
SKIP_EMULATOR_BOOT=true ./test-e2e.sh
```

**JDK 17 is required** — JDK 25 breaks Kotlin compiler. Set `JAVA_HOME` explicitly.

## Lint Commands

**You MUST run the relevant lint commands before considering any code change complete.** Fix all lint errors before committing.

```bash
# Android lint (Kotlin)
./gradlew :mobile:lintDebug -PTARGET_ABI=arm64 -PCARGO_PROFILE=release

# Rust clippy (from repo root)
cd core/src/main/rust/mihomo-android-ffi && cargo clippy -- -D warnings && cd -

# Rust format check
cd core/src/main/rust/mihomo-android-ffi && cargo fmt --check && cd -

# Flutter analyze
cd flutter_module && flutter analyze && cd -
```

Run Android lint after Kotlin changes, clippy/rustfmt after Rust changes, and flutter analyze after Dart changes.

## Architecture

Three-layer stack: **Flutter UI → Kotlin VPN Service → Rust FFI**

```
Flutter (Dart)                    MethodChannel("io.github.madeye.meow/vpn")
    ↕                             EventChannel("io.github.madeye.meow/vpn_state")
Kotlin (Android)                  EventChannel("io.github.madeye.meow/traffic")
    ↕ JNI
Rust (libmihomo_android_ffi.so)   netstack-smoltcp tun2socks + mihomo-rust engine
```

### Rust FFI (`core/src/main/rust/mihomo-android-ffi/`)

- **lib.rs**: JNI entry points (`Java_io_github_madeye_meow_core_MihomoCore_*`), engine lifecycle (tokio runtime, Tunnel, MixedListener, API server)
- **tun2socks.rs**: Reads TUN fd packets → feeds to `netstack-smoltcp` Stack → TCP connections relayed via SOCKS5 to mihomo's mixed listener on `127.0.0.1:7890`. UDP port 53 intercepted for DoH.
- **protect.rs**: Stores `JavaVM` + `GlobalRef<VpnService>`. `protect_fd(fd)` calls `VpnService.protect(int)` via JNI to prevent routing loops on proxy outbound sockets.
- **listener/**: Local copy of mihomo-rust's `MixedListener` (SOCKS5 + HTTP proxy) — copied to avoid depending on `mihomo-listener` which pulls in `tun-rs` (doesn't cross-compile to Android).
- **doh_client.rs**: DNS-over-HTTPS via reqwest through SOCKS5 proxy. Reads DoH server URLs from config, falls back to `1.1.1.1` and `8.8.8.8`.

### Patched mihomo-proxy (`core/src/main/rust/mihomo-proxy-patched/`)

Local fork of `mihomo-proxy` adding `connect.rs` with `set_pre_connect_hook()` — a global callback invoked before every outbound `connect()`. The hook calls `protect_fd()` via `socket2::Socket` (create socket → protect → connect). Applied via `[patch]` in Cargo.toml. Affects `direct.rs`, `trojan.rs`, `shadowsocks_adapter.rs`.

### Kotlin Core (`core/src/main/java/io/github/madeye/meow/`)

- **bg/BaseService.kt**: State machine (Idle→Connecting→Connected→Stopping→Stopped) with AIDL binder, RemoteCallbackList for traffic callbacks. Ported from shadowsocks-android.
- **bg/VpnService.kt**: Creates TUN interface (172.19.0.1/30, MTU 1500, route 0.0.0.0/0). Passes TUN fd + `this` (VpnService) to Rust via JNI. DNS set to 172.19.0.2 (routed through TUN → tun2socks DoH).
- **bg/MihomoInstance.kt**: Writes config.yaml (stripping `dns:` and `subscriptions:` sections), calls JNI start/stop. DNS is disabled in mihomo — handled by tun2socks DoH.
- **core/MihomoCore.kt**: JNI bridge object. `System.loadLibrary("mihomo_android_ffi")`.
- **database/**: Room database with `ClashProfile` entity (id, name, url, yamlContent, selected, lastUpdated, tx, rx).

### Flutter UI (`flutter_module/lib/`)

- **app.dart**: MaterialApp with 4-tab NavigationBar (Home, Subscribe, Traffic, Settings). `profileChanged` ValueNotifier bridges subscription changes to home screen reload.
- **services/vpn_channel.dart**: Singleton wrapping MethodChannel/EventChannel for VPN control, profile CRUD, traffic streams.
- **l10n/strings.dart**: Map-based i18n (English default, Chinese via `_Zh` subclass). Uses `S.of(context)` pattern.
- **screens/home_screen.dart**: SliverAppBar with Switch toggle, proxy node list from selected profile's YAML, status card.
- **screens/traffic_screen.dart**: Real-time speed chart (CustomPainter), session upload/download/total cards (blue/green/purple).

### Key Data Flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madeye/meow](https://github.com/madeye/meow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
