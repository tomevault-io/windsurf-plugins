---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mandacaru is an Android application that runs a lightweight Bitcoin validation node powered by Utreexo and Floresta. It bridges Rust-based Bitcoin node implementation with a modern Kotlin/Compose Android UI.

## Build & Development Commands

### Building the Project
```bash
# Debug build
./gradlew assembleDebug

# Release build
./gradlew assembleRelease

# Install debug build on connected device
./gradlew installDebug
```

### Running Tests
```bash
# Run unit tests
./gradlew test

# Run instrumented tests (requires connected device/emulator)
./gradlew connectedAndroidTest

# Run specific test class
./gradlew test --tests "com.github.jvsena42.mandacaru.SpecificTestClass"
```

### Code Quality
```bash
# Run static analysis (must pass before opening a PR)
./gradlew detekt

# Run Android Lint on the debug variant (must pass before opening a PR)
./gradlew lintDebug

# Clean build
./gradlew clean

# Check for updates
./gradlew dependencyUpdates
```

Lint configuration lives at `app/lint.xml`; detekt configuration lives at `config/detekt/detekt.yml`. When a rule fires on generated code or on an intentional project decision (e.g. ARM64-only build), prefer tuning the config over disabling the rule globally.

### Development Setup
- Requires Android 10 (API 29) minimum
- ARM64 device only (arm64-v8a) - the Rust library is compiled for ARM64 only
- Java 11 language target
- Android Studio with Compose support recommended

## Architecture

### Layer Structure (Clean Architecture + MVVM)

```
presentation/           # UI layer with Jetpack Compose
├── ui/screens/        # Screen-specific ViewModels and Composables
│   ├── main/          # MainActivity and navigation
│   ├── node/          # Node status screen
│   ├── search/        # Transaction search screen
│   └── settings/      # Settings and configuration
├── ui/components/     # Reusable Compose components
└── utils/             # UI utilities (EventFlow, notifications)

domain/                # Business logic layer
├── floresta/         # Floresta daemon integration
│   ├── FlorestaDaemon.kt        # Daemon lifecycle interface
│   ├── FlorestaDaemonImpl.kt    # Rust/FFI integration
│   ├── FlorestaService.kt       # Android foreground service
│   └── FlorestaRpcImpl.kt       # RPC client implementation
└── model/            # Domain models and RPC DTOs

data/                 # Data layer
├── FlorestaRpc.kt           # RPC interface
└── PreferencesDataSource.kt  # Preferences abstraction
```

### Key Architectural Patterns

- **MVVM**: ViewModels expose `StateFlow<UiState>` consumed by Compose UI
- **Repository Pattern**: `FlorestaRpc` interface abstracts RPC communication
- **Dependency Injection**: Koin manages singleton instances (configured in `MandacaruApplication.kt`)
- **Coroutines + Flow**: All async operations use `suspend` functions and `Flow<Result<T>>`
- **Foreground Service**: `FlorestaService` keeps the Bitcoin node alive in background

## Rust/Floresta Integration

### FFI Bridge (UniFFI)

The app communicates with Rust code via UniFFI-generated bindings:

- **Native Library**: `libuniffi_floresta.so` (ARM64 only)
- **Kotlin Bindings**: `com.florestad.florestad.kt` (auto-generated)
- **Main Classes**:
  - `Florestad`: Rust daemon wrapper with `start()`/`stop()` methods
  - `Config`: Configuration object passed to Rust (network, dataDir, etc.)
  - `Network`: Enum for Bitcoin/Testnet/Signet/Regtest

### Daemon Lifecycle

1. `FlorestaService.onStartCommand()` triggers `FlorestaDaemon.start()`
2. `FlorestaDaemonImpl.start()` reads network preference, creates `Config`, calls FFI
3. Rust daemon initializes Bitcoin node with Utreexo, binds RPC server to `127.0.0.1:<port>`
4. Android app communicates via JSON-RPC over HTTP

### Configuration by Network

Network preferences stored in SharedPreferences:
- **CURRENT_NETWORK**: "Bitcoin", "Testnet", "Signet", or "Regtest"
- **CURRENT_RPC_PORT**: Automatically set based on network (see `Constants.kt`)

Port mappings (defined in `domain/model/Constants.kt`):
- Bitcoin: 8332
- Testnet: 18332
- Signet: 38332
- Regtest: 18443

## RPC System

### JSON-RPC 2.0 Implementation

The app uses OkHttp3 to make HTTP POST requests to the Rust daemon's localhost RPC server.

**Request Format**:
```json
{
  "jsonrpc": "2.0",
  "method": "getblockchaininfo",
  "params": [],
  "id": 1
}
```

### Available RPC Methods (see `domain/model/florestaRPC/RpcMethods.kt`)

- `getblockchaininfo`: Blockchain sync status, height, Utreexo forest state
- `getpeerinfo`: Connected peer list
- `gettransaction`: Transaction lookup by txid
- `loaddescriptor`: Add wallet descriptor for address tracking
- `listdescriptors`: List loaded descriptors
- `addnode`: Connect to specific Bitcoin node
- `rescan`: Rescan blockchain for wallet transactions
- `stop`: Gracefully stop the node

### RPC Response Models

Located in `domain/model/florestaRPC/response/`:
- `GetBlockchainInfoResponse`: Contains Utreexo-specific fields (`leafCount`, `rootCount`, `rootHashes`)
- `GetTransactionResponse`: Full transaction details with inputs/outputs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jvsena42/mandacaru](https://github.com/jvsena42/mandacaru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
