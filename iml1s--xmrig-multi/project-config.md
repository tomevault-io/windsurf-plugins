---
trigger: always_on
description: This document provides comprehensive development guidelines for the XMRig Multi cross-platform mining application. Follow these guidelines to maintain code quality, consistency, and architectural integrity across Android, iOS, Web, Desktop, WearOS, and watchOS platforms.
---

# AGENTS.md - Development Guidelines for XMRig Multi

This document provides comprehensive development guidelines for the XMRig Multi cross-platform mining application. Follow these guidelines to maintain code quality, consistency, and architectural integrity across Android, iOS, Web, Desktop, WearOS, and watchOS platforms.

Human-facing build and platform notes live in [docs/README.md](docs/README.md).

## Build Commands

### Android
```bash
# Debug APK build
./gradlew :app:assembleDebug

# Release APK build
./gradlew :app:assembleRelease

# Unit tests
./gradlew testDebugUnitTest

# Lint code analysis
./gradlew lintDebug

# Clean build
./gradlew clean

# Run single test class
./gradlew testDebugUnitTest --tests "*MiningConfigTest*"

# Run single test method
./gradlew testDebugUnitTest --tests "*MiningConfigTest*isValid*"
```

### iOS
```bash
# From the repository root. Needs ios-cmake + libuv.a; see docs/ios.md.
# Tracked libxmrig-ios-arm64.a is upstream donate, not this repo's fee wallet.
./ios/XMRigCore/scripts/build-ios.sh
open ios/XMRigMiner-iOS.xcodeproj
```

### Web Miner
```bash
# Install dependencies
cd web && npm install

# Development server (port 5173)
cd web && npm run dev

# Build production
cd web && npm run build

# Preview production build
cd web && npm run preview

# WebSocket proxy for mining
cd web/proxy && node server.js
```

### Desktop (Tauri)
```bash
# Install dependencies
cd desktop && npm install

# Development
npm run tauri:dev

# Production build
npm run tauri:build
```

### WearOS
```bash
# Build debug APK
./gradlew :wearos:assembleDebug
```

### watchOS
```bash
# Open Xcode project
cd watchos && open XMRigWatch.xcodeproj
```

## Test Commands

### Android Unit Tests
```bash
# All unit tests
./gradlew testDebugUnitTest

# Single test class
./gradlew testDebugUnitTest --tests "*MiningConfigTest*"

# Test method pattern
./gradlew testDebugUnitTest --tests "*MiningConfigTest*isValid*"

# With coverage
./gradlew testDebugUnitTest jacocoTestReport
```

### Android Instrumentation Tests
```bash
# Connected device tests
./gradlew connectedAndroidTest

# Specific device
./gradlew connectedDebugAndroidTest -Pandroid.testInstrumentationRunnerArguments.class=com.iml1s.xmrigminer.ExampleInstrumentedTest
```

## Code Style Guidelines

### Kotlin (Android)

#### Architecture Patterns
- **MVI Pattern**: Use sealed interfaces for `State`, `Event`, and `Effect`
- **Clean Architecture**: Separate data/presentation/service layers
- **Repository Pattern**: Abstract data sources behind repository interfaces
- **Dependency Injection**: Use Hilt for all dependencies

#### Naming Conventions
- Classes: `PascalCase` (e.g., `MiningConfig`, `ConfigViewModel`)
- Functions/Methods: `camelCase` (e.g., `isValid()`, `startMining()`)
- Variables/Properties: `camelCase` (e.g., `walletAddress`, `poolUrl`)
- Constants: `SCREAMING_SNAKE_CASE` (e.g., `DEFAULT_POOL_URL`)
- Test Methods: Backtick enclosed descriptive names (e.g., `\`isValid returns true for valid config\``)

#### Code Structure
```kotlin
// Data class with defaults
data class MiningConfig(
    val poolUrl: String = "pool.supportxmr.com:3333",
    val walletAddress: String = "",
    val threads: Int = Runtime.getRuntime().availableProcessors() - 1
)

// MVI Contract
sealed interface ConfigUiState {
    data object Loading : ConfigUiState
    data class Success(val config: MiningConfig) : ConfigUiState
    data class Error(val message: String) : ConfigUiState
}

sealed interface ConfigUiEvent {
    data class PoolSelected(val pool: Pool) : ConfigUiEvent
    data object SaveConfig : ConfigUiEvent
}

// ViewModel
class ConfigViewModel @Inject constructor(
    private val repository: ConfigRepository
) : ViewModel() {
    val state: StateFlow<ConfigUiState> = // ...
    
    fun onEvent(event: ConfigUiEvent) {
        // Handle events
    }
}
```

#### Imports Organization
- AndroidX imports first
- Third-party libraries (Hilt, Compose, etc.)
- Standard library imports
- Group related imports together

#### Error Handling
- Use `Result<T>` for synchronous operations
- Use `Flow<T>` with error states for async operations
- Validate input at domain boundaries
- Provide meaningful error messages to users

### JavaScript (Web)

#### Architecture Patterns
- ES6 modules with clear import/export structure
- Class-based components with single responsibility
- Event-driven architecture for UI interactions
- Separation of concerns (UI logic, mining logic, validation)

#### Naming Conventions
- Classes: `PascalCase` (e.g., `App`, `Miner`)
- Methods: `camelCase` (e.g., `startMining()`, `validateAddress()`)
- Variables: `camelCase` (e.g., `walletAddress`, `miningStats`)
- Constants: `SCREAMING_SNAKE_CASE` (e.g., `DEFAULT_POOL_URL`)
- DOM elements: Prefixed with `dom` (e.g., `dom.startBtn`)

#### Code Structure
```javascript
// Class-based architecture
class App {
    constructor() {
        this.miner = new Miner();
        this.init();
    }

    init() {
        this.bindEvents();
        this.loadSettings();
    }

    bindEvents() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ImL1s/xmrig-multi](https://github.com/ImL1s/xmrig-multi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
