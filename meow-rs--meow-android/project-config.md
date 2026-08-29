---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
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
cd core/src/main/rust/meow-android-ffi && cargo clippy -- -D warnings && cd -

# Rust format check
cd core/src/main/rust/meow-android-ffi && cargo fmt --check && cd -

# Unit tests (engine API client + en/zh string parity)
./gradlew :core:testDebugUnitTest :mobile:testDebugUnitTest
```

Run Android lint and the unit tests after Kotlin changes, clippy/rustfmt after Rust changes.

## Architecture

Three-layer stack: **Compose UI → Kotlin VPN Service → Rust FFI**

```
Compose (Kotlin)                  ViewModels ← StateFlow ← VpnStateRepository
    ↕ direct calls                                          (AIDL callbacks)
Kotlin (Android)                  MeowApi (OkHttp) → 127.0.0.1:9090
    ↕ JNI
Rust (libmeow_android_ffi.so)   lwip netstack tun2socks + meow-rs engine
```

The UI runs in the same process as the Kotlin layer and calls it directly —
there is no serialization boundary between them. The one remote hop is the
engine's own controller API on loopback, which crosses into `:vpn`.

### Rust FFI (`core/src/main/rust/meow-android-ffi/`)

- **lib.rs**: JNI entry points (`Java_io_github_madeye_meow_core_MeowCore_*`), engine lifecycle (tokio runtime, Tunnel, API server). No SOCKS5/HTTP loopback listener — every TUN flow is dispatched in-process.
- **tun2socks.rs**: Reads TUN fd packets. UDP/53 is intercepted pre-stack: A/AAAA queries are answered in-process by the engine's `meow_dns::Resolver` (`DnsServer::handle_query`), other qtypes are forwarded to upstream DNS over protected sockets. Everything else feeds the `lwip` netstack — each accepted TCP flow is wrapped as `NetstackConn` (a `ProxyConn` newtype around `lwip::TcpStream`) and handed straight to `meow_tunnel::tcp::handle_tcp(&inner, conn, metadata)`; UDP flows are dispatched to `meow_tunnel::udp::handle_udp` with per-session reply readers.
- **protect.rs**: Implements `meow_common::SocketProtector` via a JNI shim around `VpnService.protect(int)`. Installed once in `nativeStartTun2Socks`; meow-rs invokes it for every outbound TCP/UDP fd (proxy adapters + the DNS resolver's default `SocketFactory`) before `connect()`/`bind()`.
- **engine.rs**: `tunnel()` accessor — returns the running `Tunnel` handle so `tun2socks` can dispatch flows through `meow_tunnel::{tcp,udp}` without re-implementing rule routing. Also `strip_and_inject`: strips listener ports, `sniffer:`, and the user `dns:` block from config.yaml and injects the pinned fake-IP DNS block (same pattern as meow-ios).
- **diagnostics.rs**: JNI-exposed native connectivity probes (`nativeTestDirectTcp`, UDP) used by the Settings diagnostics UI.
- **logging.rs**: `android_logger` / tracing setup bridging Rust logs to logcat and the in-app log stream.

### Kotlin Core (`core/src/main/java/io/github/madeye/meow/`)

- **bg/BaseService.kt**: State machine (Idle→Connecting→Connected→Stopping→Stopped) with AIDL binder, RemoteCallbackList for traffic callbacks. Ported from shadowsocks-android.
- **bg/VpnService.kt**: Creates TUN interface (172.19.0.1/30, MTU 1500, route 0.0.0.0/0). Passes TUN fd + `this` (VpnService) to Rust via JNI. DNS set to 172.19.0.2 (routed through TUN → in-process DNS interception in tun2socks).
- **bg/MeowInstance.kt**: Writes config.yaml (stripping only the app-managed `subscriptions:` block — `dns:`/listeners/`sniffer:` are handled by `engine::strip_and_inject` on the Rust side), calls JNI start/stop.
- **core/MeowCore.kt**: JNI bridge object. `System.loadLibrary("meow_android_ffi")`.
- **database/**: Room database with `ClashProfile` entity (id, name, url, yamlContent, selected, lastUpdated, tx, rx).

### Engine API client (`core/src/main/java/io/github/madeye/meow/api/`)

- **MeowApi.kt**: OkHttp + kotlinx-serialization client for the embedded engine's
  Clash-compatible controller (`http://127.0.0.1:9090`, started by `MeowInstance`).
  Proxies/groups, delay probes, rules, connections, configs, and the `/logs`
  websocket (500ms→30s reconnect ramp). `baseUrl` is injectable for tests.
- **MeowApiModels.kt**: `/proxies` is a heterogeneous map discriminated by a field

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meow-rs/meow-android](https://github.com/meow-rs/meow-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
