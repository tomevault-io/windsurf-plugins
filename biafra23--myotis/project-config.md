---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
# Build all modules. NOTE: :android-app builds the Rust engine from source by
# default, so a full build needs the Android Rust toolchain (cargo + cargo-ndk +
# NDK r28+ + the aarch64/x86_64-linux-android rustup targets). Without it, add
# -PskipRustEngine to build the app on the Java engine (see the Rust section below).
./gradlew build                  # add -PskipRustEngine without the Android Rust toolchain

# Compile only (no tests)
./gradlew compileJava

# Run tests (all modules) — same toolchain note as `build`; :android-app's test
# tasks also go through its preBuild gate.
./gradlew test                   # add -PskipRustEngine without the Android Rust toolchain

# Run a single test class
./gradlew :networking:test --tests "com.jaeckel.ethp2p.networking.rlpx.HandshakeRoundTripTest"

# Start daemon (mainnet, blocks until stopped)
./gradlew :app:run

# Start daemon on another network
./gradlew :app:run -Pnetwork=gnosis     # Gnosis Chain (chainId 100, its own beacon chain)
./gradlew :app:run -Pnetwork=sepolia    # Ethereum testnet
# (holesky was retired — the EF shut it down in Oct 2025: no peers, no checkpoint servers)

# Run a second network alongside mainnet (separate daemon, separate port + socket)
./gradlew :app:run -Pnetwork=gnosis -Pport=30304

# Send IPC commands to running daemon
./gradlew :app:run -Pargs=status
./gradlew :app:run -Pargs=peers
./gradlew :app:run -Pargs="get-headers 21000000 3"
./gradlew :app:run -Pargs=stop
./gradlew :app:run -Pargs=purge-cache

# Rust workspace (rust/ — native BLS + the growing Rust engine). OPTIONAL for the
# JVM hosts (daemon/desktop): without cargo these self-skip with one note and the
# pure-Java build is unaffected. The ANDROID app is the exception — it requires the
# toolchain by default (see below), or -PskipRustEngine to opt out.
./gradlew cargoBuildHost   # cargo build --release (auto-runs before :app:run / :consensus:test)
./gradlew cargoTest        # cargo test --workspace (part of `check`)
./gradlew cargoNdkAndroid  # Android jniLibs, built from source (needs cargo-ndk + NDK + Android rustup targets)

# The Android app builds the Rust engine FROM SOURCE by default — cargo + cargo-ndk
# + NDK r28+ + `rustup target add aarch64-linux-android x86_64-linux-android` are
# REQUIRED to build :android-app. There is no committed .so (so nothing can drift);
# an Android build also regenerates the committed UniFFI bindings (:myotis-engines)
# from source — other workflows do NOT, so regenerate them explicitly with
# `./gradlew uniffiGenerateKotlin` after changing the Rust FFI. Opt out when you
# lack the toolchain — the build tells you about the switch — with:
./gradlew :android-app:assembleDebug -PskipRustEngine  # Java engine only (no Rust engine / native BLS)

# iOS (macOS only; needs Xcode 26+ and the rustup targets on the toolchain the
# workspace's rust-toolchain.toml selects — i.e.
# `rustup target add --toolchain stable aarch64-apple-ios aarch64-apple-ios-sim`).
# Cargo stays optional here too: without the toolchain (and with no previously
# built libmyotis_engine.a) :app-ios disables its framework tasks with a warning
# instead of failing the build — the full toolchain is required only to actually
# build the iOS app.
./gradlew cargoBuildIosSim                              # libmyotis_engine.a for the arm64 simulator
./gradlew cargoBuildIosDevice                           # libmyotis_engine.a for arm64 devices
./gradlew :app-ios:linkDebugFrameworkIosSimulatorArm64  # MyotisKit.framework (runs cargo first)
# The app itself builds from ios-app/ (Xcode project, regenerable with `xcodegen generate`):
#   cd ios-app && xcodebuild -project Myotis.xcodeproj -scheme Myotis \
#     -destination 'platform=iOS Simulator,name=<device>' build
```

## Architecture

Key Gradle modules:

- **myotis-engines** — the engine SELECTOR (`Engines`/`SelectorEngine`/`RustMyotisEngine`):
  hosts' composition roots call `Engines.engine()`; `myotis.engine=java|rust|auto` routes
  each network (re)start to the Java engine or the Rust one (`rust/myotis-engine`,
  UniFFI-generated Kotlin bindings over JNA — committed in this module, regenerated
  via `uniffiGenerateKotlin`; compound values as JSON pinned by golden tests both sides).
- **myotis-api** — THE ENGINE CONTRACT: zero-dependency Java-17 interfaces
  (`io.myotis.api` + `io.myotis.api.ports`) every host consumes exclusively.
  FFI-portable types only (byte[], String, long, double[], enums, flat records;
  blocking methods). Implemented by **node-core**'s `io.myotis.node.api` adapters
  (`JavaMyotisEngine`/`JavaChainHandle` over `ChainStack`/`NodeRegistry`). Designed so a Rust engine can replace the Java one behind the same
  surface — see docs/reimplementation/05-engine-api-bindings.md.
- **core** — Cryptographic identity (`NodeKey`), data types (`BlockHeader`), ENR decoding
- **networking** — Three protocol layers, all Netty-based:
  - `discv4` — UDP peer discovery using Kademlia DHT (ping/pong/findnode/neighbors)
  - `rlpx` — TCP transport: EIP-8 ECIES handshake → AES-256-CTR framed channel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [biafra23/myotis](https://github.com/biafra23/myotis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
