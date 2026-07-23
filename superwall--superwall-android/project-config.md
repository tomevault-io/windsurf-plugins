---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Superwall Android SDK - an open-source framework for remotely configuring paywalls and managing subscriptions in Android apps. The SDK provides a WebView-based paywall system with server-side configuration, A/B testing, and comprehensive analytics.

## Development Commands

### Building and Testing
```bash
# Build the project
./gradlew build

# Run unit tests
./gradlew test

# Run instrumentation tests (requires connected device/emulator)
./gradlew :app:connectedCheck

# Run integration tests with screenshot recording
./gradlew recordDebugAndroidTestScreenshots

# Build and publish SDK locally
./gradlew publishToMavenLocal

# Generate build info
./gradlew generateBuildInfo
```

### Code Quality
```bash
# Run linting (if available)
./gradlew lint

# Run code formatting (if available)
./gradlew ktlintFormat
```

### Testing
- Unit tests: `./gradlew :superwall:test`
- Integration tests: `./gradlew :app:connectedCheck`
- Screenshot tests use Dropshot library for UI regression testing
- Tests require API 33 Pixel 7 emulator for consistent screenshot results

## Architecture Overview

### Core Components

**Main SDK Entry Point**: `superwall/src/main/java/com/superwall/sdk/Superwall.kt`
- Singleton configuration through `Superwall.configure()`
- Requires `Application`, `apiKey`, optional `PurchaseController`, `SuperwallOptions`, `ActivityProvider`
- Creates `DependencyContainer` which orchestrates all components

**Dependency Container**: `superwall/src/main/java/com/superwall/sdk/dependencies/DependencyContainer.kt`
- Central dependency injection container
- Implements multiple factory interfaces for clean separation
- Manages component lifecycle and inter-dependencies

**Key Managers**:
- `ConfigManager`: Fetches and caches remote configuration with retry logic
- `IdentityManager`: Handles user identity, attributes, and anonymous users
- `StoreManager`: Manages Google Play Billing integration and product fetching
- `PaywallManager`: Handles paywall presentation, caching, and lifecycle

### Paywall System

**Presentation Flow**:
1. `Superwall.register()` - Registers placement events
2. `TrackingLogic.canTriggerPaywall()` - Evaluates trigger conditions
3. `PaywallPresentationHandler` - Manages presentation logic
4. `PaywallView` - Custom FrameLayout hosting WebView-based paywalls

**Rule Evaluation**: `superwall/src/main/java/com/superwall/sdk/paywall/presentation/rule_logic/`
- CEL (Common Expression Language) evaluator for complex rules
- JavaScript fallback for rule evaluation
- User/device attribute matching for targeting

### Billing Integration

**Transaction Management**: `superwall/src/main/java/com/superwall/sdk/store/transactions/TransactionManager.kt`
- Supports internal paywall purchases, external purchases, and observer mode
- Google Play Billing wrapper with connection lifecycle management
- Purchase state tracking and restoration functionality

**Purchase Sources**:
- `Internal`: Purchases from within paywalls
- `ExternalPurchase`: Direct API purchases
- `ObserverMode`: Tracking external purchases

### Storage and Caching

**Local Storage**: `superwall/src/main/java/com/superwall/sdk/storage/LocalStorage.kt`
- SharedPreferences for configuration caching
- SQLite (Room) for complex data storage
- Multi-layer caching strategy (memory, disk, network)

**Cache Types**:
- Config caching with refresh mechanisms
- Paywall content and asset caching
- Product caching to reduce API calls
- A/B test assignment persistence

### Analytics and Tracking

**Event System**: `superwall/src/main/java/com/superwall/sdk/analytics/internal/Tracking.kt`
- Comprehensive event tracking with internal and user-initiated events
- Session management and device classification
- Event queuing for batch network transmission
- Delegate notification system for custom event handling

## Key Extension Points

### Delegate System
**SuperwallDelegate**: Primary interface for SDK events and lifecycle callbacks
- `handleSuperwallEvent()`: All Superwall events
- `subscriptionStatusDidChange()`: Subscription state changes
- `willPresentPaywall()` / `didPresentPaywall()`: Paywall lifecycle
- `handleCustomPaywallAction()`: Custom paywall actions

### Purchase Controller
**PurchaseController**: Custom purchase handling interface
- `purchase()`: Handle purchase requests
- `restorePurchases()`: Handle restoration
- Supports both automatic and manual purchase flows

### Public API
- `Superwall.register()`: Register placement events
- `Superwall.identify()`: User identification
- `Superwall.setUserAttributes()`: User property management
- `Superwall.purchase()`: Direct purchase API
- `Superwall.restorePurchases()`: Restore functionality

## Module Structure

- **superwall/**: Main SDK module
- **superwall-compose/**: Jetpack Compose integration
- **app/**: Demo app with integration tests
- **example/**: Example implementations with different configurations
- **buildSrc/**: Build configuration and scripts

## Development Notes

### Thread Safety
- Extensive use of Kotlin Coroutines with proper scoping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superwall/Superwall-Android](https://github.com/superwall/Superwall-Android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
