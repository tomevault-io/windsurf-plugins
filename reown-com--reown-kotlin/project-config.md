---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## AI Skills

When writing or modifying Kotlin code in this repository, use the **kotlin-coding** skill located at:
- `.claude/skills/kotlin-coding/SKILL.md` - Core patterns and validation checklist
- `.claude/skills/kotlin-coding/REFERENCE.md` - Extended patterns and examples

These skills encode project-specific conventions including use case patterns, Koin DI setup, coroutines usage, and data modeling.

## Project Overview

**reown-kotlin** is the official Kotlin/Android SDK for WalletConnect and Reown protocols. It enables:

- **Wallets**: Session management, transaction signing, push notifications
- **dApps**: Wallet connections, session requests, blockchain interactions
- **Payments**: Crypto payment links via WalletConnect Pay

**Tech Stack:**
- Kotlin 2.2.0, JVM 11, Min SDK 23, Target SDK 35
- Koin (DI), Moshi + KSP (JSON), SQLDelight (DB)
- Retrofit + OkHttp (HTTP), Scarlet (WebSocket)
- Coroutines 1.10.2, Jetpack Compose (UI)

**License:** WalletConnect Community License

## Repository Structure

```
reown-kotlin/
├── foundation/              # Pure Kotlin/JVM - crypto, JWT, HTTP utilities
├── core/
│   ├── android/             # Android core - relay, pairing, verification
│   ├── modal/               # Shared modal UI components
│   └── bom/                 # Bill of Materials for version management
├── protocol/
│   ├── sign/                # WalletConnect Sign protocol implementation
│   └── notify/              # Notification protocol implementation
├── product/
│   ├── walletkit/           # High-level wallet SDK
│   ├── appkit/              # High-level dApp SDK with Compose UI
│   ├── pay/                 # Payment SDK (standalone, uses Rust/UniFFI)
│   └── pos/                 # Point of Sale application
├── sample/
│   ├── wallet/              # Wallet reference implementation
│   ├── dapp/                # dApp reference implementation
│   ├── modal/               # Modal integration example
│   └── pos/                 # POS example
├── buildSrc/                # Custom Gradle plugins and build logic
├── gradle/
│   ├── libs.versions.toml   # Version catalog (dependencies)
│   ├── proguard-rules/      # ProGuard configurations
│   └── consumer-rules/      # Consumer ProGuard rules
├── .claude/skills/          # AI agent skills for this project
└── docs/                    # Documentation files
```

## Key Commands

### Build Commands

```bash
# Build all modules
./gradlew build

# Build specific module
./gradlew :core:android:build
./gradlew :protocol:sign:build
./gradlew :product:walletkit:build

# Clean build
./gradlew clean
```

### Testing

```bash
# Run all unit tests
./gradlew test

# Run unit tests for specific module
./gradlew :protocol:sign:testDebugUnitTest

# Run single test class
./gradlew :protocol:sign:testDebugUnitTest --tests "com.reown.sign.SomeTest"

# Run single test method
./gradlew :protocol:sign:testDebugUnitTest --tests "com.reown.sign.SomeTest.testMethod"

# Run instrumented tests (requires emulator/device)
./gradlew connectedAndroidTest
```

### E2E Tests (Maestro)

The wallet sample has Maestro-based E2E tests for the WalletConnect Pay flow.

**Prerequisites:**
- [Maestro CLI](https://maestro.mobile.dev/) installed
- Android emulator running
- Wallet app built with `ENABLE_TEST_MODE=true` (enables the URL input field for test automation)

**Setup:**

```bash
# 1. Download test flows from WalletConnect/actions repo
./scripts/setup-maestro-pay-tests.sh

# 2. Create .env.maestro from the example and fill in merchant credentials
cp .env.maestro.example .env.maestro

# 3. Build and install the wallet app with test mode enabled
ENABLE_TEST_MODE=true ./gradlew :sample:wallet:assembleDebug
adb install sample/wallet/build/outputs/apk/debug/*.apk
```

**Running tests:**

```bash
# Run all Pay E2E tests
APP_ID=com.reown.sample.wallet.debug ./scripts/run-maestro-pay-tests.sh

# Run a specific test file
maestro test --env APP_ID=com.reown.sample.wallet.debug .maestro/pay_single_option_nokyc.yaml
```

**`ENABLE_TEST_MODE`:** This env var controls whether the manual URL input field is shown in the scanner screen. It defaults to `false` so the field is hidden in all builds (debug, internal, release). Only CI E2E builds and local test runs should set it to `true`.

### Code Quality

```bash
# Run lint
./gradlew lint

# Generate coverage report (JaCoCo)
./gradlew jacocoTestReport
```

### Sample Apps

```bash
# Build sample apps
./gradlew :sample:wallet:assembleDebug
./gradlew :sample:dapp:assembleDebug
./gradlew :sample:modal:assembleDebug
./gradlew :sample:pos:assembleDebug
```

Sample apps auto-generate mock `google-services.json` and `secrets.properties` files on first build if they are missing, so no manual setup is needed for debug builds.

## Architecture Overview

### Layer Structure

The SDK follows a four-layer architecture where each layer depends only on layers below it:

```
┌─────────────────────────────────────────────────────────┐
│  PRODUCT LAYER                                          │
│  product/walletkit/  - High-level wallet SDK            │
│  product/appkit/     - High-level dApp SDK (Compose UI) │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reown-com/reown-kotlin](https://github.com/reown-com/reown-kotlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
