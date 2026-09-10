---
trigger: always_on
description: > Context and conventions for AI coding assistants working in **XrayFA**. Humans start with `README.md`. Keep this file in the same PR as any change that affects build, modules, versions, CI, or conventions (see §11).
---

# AGENT.md

> Context and conventions for AI coding assistants working in **XrayFA**. Humans start with `README.md`. Keep this file in the same PR as any change that affects build, modules, versions, CI, or conventions (see §11).

---

## 1. Project Overview

**XrayFA** is a **Kotlin Multiplatform VPN/proxy client** for **Android and iOS**, built on [Xray-core](https://github.com/XTLS/Xray-core). Protocols: VLESS, VMess, Shadowsocks, Trojan, SOCKS, HTTP, Hysteria2, and others.

- **UI**: Compose Multiplatform shared `RootContent` (Decompose Config\|Home pager + overlays). Android supplies `AndroidPlatformRootHooks`; iOS supplies `IosPlatformRootHooks` (开发中 for remaining gaps — see `docs/IOS_STUBS.md`).
- **Logic**: Decompose components + Koin 4.0.1 (not Dagger)
- **Data**: Room KMP + DataStore KMP; repositories in `:core:data`
- **VPN**: Android `VpnService`; iOS Network Extension (`NEPacketTunnelProvider`)
- **Core**: Go + gomobile → `libv2ray.aar` / `LibXrayLite.xcframework`
- **TUN**: C `hev-socks5-tunnel` (Android JNI; iOS xcframework)
- **Distribution**: GitHub Releases, F-Droid (`com.android.xrayfa`). Google Play is not planned (`APPLICATION_ID_PLAY` kept but unused)
- **License**: Apache-2.0
- **Version**: `VERSION_NAME` / `VERSION_CODE` in `gradle.properties` (currently 1.7.0 / 34)

Product rule: **Android is the reference; iOS aligns to Android.** Do not add a second parallel implementation of a screen.

**KMP migration is closed (Phase 9).** New features go in `:shared` + `PlatformRootHooks` only. iOS gaps iterate via `docs/KMP_POST_MIGRATION.md` / `docs/IOS_STUBS.md`.

---

## 2. Prerequisites

| Tool | Source of truth |
|------|-----------------|
| JDK | 11 bytecode / **17** to run Gradle (CI uses Temurin 17) |
| Android SDK | compileSdk **36**, minSdk **28**, targetSdk **36** |
| NDK | **28.2.13676358** (CI: r28c) for `:tun2socks` |
| Go | `AndroidLibXrayLite/go.mod` |
| gomobile / gobind | `go install golang.org/x/mobile/cmd/{gomobile,gobind}@latest` |
| Gradle | Wrapper **8.11.1** — always `./gradlew` |
| Xcode | Needed for `iosApp` / Network Extension |

Catalog versions live **only** in `gradle/libs.versions.toml` (Kotlin **2.1.10**, KSP **2.1.10-1.0.31**, AGP **8.10.0**, Compose BOM **2026.03.00**, CMP **1.7.3**, Room **2.7.0**, Koin **4.0.1**, Decompose **3.2.2**). Do not hardcode versions in `build.gradle.kts` (LeakCanary in `:androidApp` debug is the listed exception).

---

## 3. Build & Run

### 3.1 Clone

```bash
git clone --recursive <repo-url>
cd XrayFA
git submodule update --init --recursive   # if already cloned
```

Submodules: `AndroidLibXrayLite/` (Xray-core gomobile), `tun2socks/src/main/jni/hev-socks5-tunnel/`.

### 3.2 Android native core (`libv2ray.aar`)

`:androidApp` needs `androidApp/libs/libv2ray.aar` (gitignored). Generate once:

```bash
./gradlew copyXrayLib
# or: cd AndroidLibXrayLite && gomobile bind … && cp libv2ray.aar ../androidApp/libs/
```

`preBuild` does **not** depend on `copyXrayLib` (intentional).

### 3.3 iOS native core (`LibXrayLite.xcframework`)

Required before `:shared` / `:core:native-bridge` iOS compile:

```bash
./scripts/build_libxray_ios.sh
```

Output: `AndroidLibXrayLite/LibXrayLite.xcframework` (gitignored). CI caches it in `ios-shared.yml`.

### 3.4 Gradle

```bash
./gradlew assembleDebug
./gradlew assembleRelease          # minify + shrink; unsigned if no keystore
./gradlew :androidApp:compileDebugKotlin
./gradlew :shared:compileDebugKotlin
```

- Signing: `KEYSTORE_PASSWORD` / `KEY_ALIAS` / `KEY_PASSWORD` + `androidApp/xrayfa.jks`
- Override applicationId with `-PAPPLICATION_ID=`
- Debug APK is **debuggable**, no R8, and includes **LeakCanary** — expect jank vs release

### 3.5 Windows

`:tun2socks` runs `fix_headers.bat` before native `preBuild`. Run it in the repo root if header includes fail.

---

## 4. Testing

```bash
./gradlew :common:testDebugUnitTest
./gradlew :domain:testDebugUnitTest          # parser goldens + kotlinx JSON + Agent catalog
./gradlew :core:datastore:testDebugUnitTest
./gradlew :domain:iosX64Test                 # Native stand-in on Intel Macs
./gradlew :domain:iosSimulatorArm64Test      # Apple Silicon simulator
./gradlew test                               # JVM/Android unit tests (needs aar for some modules)
```

- Shared business logic belongs in **`commonTest`**, not `androidUnitTest`. Gson parity stays on Android (Gson is JVM-only).
- Parser / config goldens: `domain/src/commonTest/kotlin/com/android/xrayfa/parser/` (`ProtocolParserGoldenTest`, `AbstractConfigParserGoldenTest`).
- Agent catalog: `domain/src/commonTest/kotlin/com/android/xrayfa/agent/XrayAgentCatalogTest.kt` (node/subscription summaries must not leak URLs or node JSON).
- Native delay mapping: `core/native-bridge/.../DecodeNativeDelayMsTest.kt`.
- GeoLite country flags: `common/src/commonTest/.../CountryFlagEmojiTest.kt`, `GeoIpCountryDisplayTest.kt`, `MmdbCountryLookupTest.kt` (MaxMind `GeoIP2-Country-Test.mmdb` fixture in `androidUnitTest/resources`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Q7DF1/XrayFA](https://github.com/Q7DF1/XrayFA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
