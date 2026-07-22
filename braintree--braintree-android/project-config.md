---
trigger: always_on
description: The Braintree Android SDK is a multi-module Kotlin library that enables merchants to accept payments (credit card, PayPal, Venmo, Google Pay, 3D Secure, and more) in Android apps. It is distributed via Maven Central as individual AAR artifacts under the `com.braintreepayments.api` group.
---

# Braintree Android SDK — Claude Code Context

## Project Overview

The Braintree Android SDK is a multi-module Kotlin library that enables merchants to accept payments (credit card, PayPal, Venmo, Google Pay, 3D Secure, and more) in Android apps. It is distributed via Maven Central as individual AAR artifacts under the `com.braintreepayments.api` group.

- **Minimum Android SDK**: API 23
- **Target/Compile SDK**: API 36
- **Kotlin version**: 1.9.10
- **Java version**: 17
- **Gradle version**: 8.9.1 (via wrapper)
- **Current active major version**: v5.x

## Repository Structure

```
BraintreeCore/          # Core networking, config, auth, analytics
Card/                   # Credit/debit card tokenization
PayPal/                 # PayPal checkout and vault flows
Venmo/                  # Venmo app switch integration
GooglePay/              # Google Pay integration
ThreeDSecure/           # 3D Secure via Cardinal SDK
AmericanExpress/        # AmEx rewards balance
DataCollector/          # Fraud detection (Kount)
LocalPayment/           # Local/alternative payment methods
SEPADirectDebit/        # SEPA Direct Debit payments
ShopperInsights/        # Shopper enrichment data
PayPalMessaging/        # Pay Later messaging UI
UIComponents/           # Branded PayPal/Venmo buttons with full flows
SharedUtils/            # Internal HTTP, validation, URL utilities
TestUtils/              # Shared mock builders and test fixtures
Demo/                   # Reference implementation app
gradle/                 # Version catalog (libs.versions.toml) and wrapper
detekt/                 # Static analysis configuration
.github/workflows/      # GitHub Actions CI workflows
```

## Build & Development Setup

```bash
# Build the whole project
./gradlew build

# Build a specific module
./gradlew :Card:build

# Generate API docs (Dokka)
./gradlew dokkaHtmlMultiModule
```

Open the project root in Android Studio. No separate setup step is needed — Gradle handles all dependency resolution.

## Running Tests

```bash
# All unit tests
./gradlew --continue clean testRelease

# Unit tests for a specific module
./gradlew :Card:testRelease

# Instrumentation tests (requires connected device or emulator)
./gradlew --continue connectedAndroidTest

# Lint + Detekt (must pass before merging)
./ci lint
```

CI uses the `./ci` shell script as a wrapper. The three subcommands are `unit_tests`, `integration_tests`, and `lint`.

Instrumentation tests run on API 23, 31, and 35 emulators (x86_64, Pixel 7 Pro profile).

## Architecture & Key Patterns

### BraintreeClient as Central Hub
All payment method clients accept a `BraintreeClient` instance in their constructor. `BraintreeClient` owns:
- HTTP and GraphQL networking
- Authorization management (tokenization key vs. client token)
- Configuration loading and caching
- Analytics
- Browser switch coordination

Do not duplicate any of these concerns in individual payment clients.

### Client / Callback Split
Every public payment method module exposes a `[Payment]Client` class. Public methods accept a functional interface callback:

```kotlin
// Public API (callback-based)
fun tokenize(card: Card, callback: CardTokenizeCallback)

// Internal implementation (coroutine-based)
suspend fun tokenize(card: Card): CardResult
```

Implement the suspend function as the primary path; wrap it in a `coroutineScope.launch { }` block inside the callback method. This allows both styles to coexist without duplicating logic.

### Sealed Class Results
All async operations return a sealed result class — never throw exceptions into callback paths:

```kotlin
sealed class CardResult {
    class Success(val nonce: CardNonce) : CardResult()
    class Failure(val error: Exception) : CardResult()
}

// Some flows also have a Cancel state:
sealed class PayPalResult {
    class Success(val nonce: PayPalAccountNonce) : PayPalResult()
    class Failure(val error: Exception) : PayPalResult()
    data object Cancel : PayPalResult()
}
```

### Constructor Overloading for Testability
Public constructors take only `(Context, authorization: String)`. Internal constructors accept all dependencies for injection during testing:

```kotlin
// Public
constructor(context: Context, authorization: String) :
    this(BraintreeClient(context, authorization))

// Internal (full DI)
internal constructor(
    braintreeClient: BraintreeClient,
    apiClient: CardApiClient,
    analyticsParamRepository: AnalyticsParamRepository,
    dispatcher: CoroutineDispatcher,
    coroutineScope: CoroutineScope
)
```

Always provide both constructors when adding a new client.

### Parcelable Nonces
All `PaymentMethodNonce` subclasses (`CardNonce`, `PayPalAccountNonce`, etc.) implement `Parcelable` via the Kotlin Parcelize plugin. This is required for IPC across Activity boundaries.

### Analytics
Each client tracks events using constants on the module's analytics object (e.g., `CardAnalytics.CARD_TOKENIZE_STARTED`). Call `braintreeClient.sendAnalyticsEvent(...)` on every success/failure path.

### Configuration Management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [braintree/braintree_android](https://github.com/braintree/braintree_android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
