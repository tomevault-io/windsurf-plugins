---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Clerk Android SDK is a modular authentication SDK for Android applications. It provides two main artifacts:
- **`clerk-android-api`** (`source/api`) - Core API and authentication logic
- **`clerk-android-ui`** (`source/ui`) - Prebuilt Jetpack Compose UI components (includes API)

The SDK enables user management with sign-up, sign-in, MFA, passkeys, OAuth/SSO, and profile management.

## Common Development Commands

### Building

```bash
# Build the entire project
./gradlew build

# Build specific modules
./gradlew :source:api:build
./gradlew :source:ui:build

# Assemble without tests
./gradlew assemble

# Verify Maven publishing metadata locally
./gradlew :source:api:publishToMavenLocal
```

### Testing

```bash
# Run all tests
./gradlew test

# Run tests for specific module
./gradlew :source:api:test
./gradlew :source:ui:test

# Run a single test class
./gradlew :source:api:test --tests "com.clerk.sdk.SpecificTest"

# Run UI snapshot tests (Paparazzi)
./gradlew :source:ui:testDebug
./gradlew :source:ui:recordPaparazziDebug  # Update snapshots

# Run Android instrumentation tests
./gradlew connectedAndroidTest
./gradlew :source:api:connectedDebugAndroidTest
```

### Code Quality

```bash
# Format code (must pass before commit)
./gradlew spotlessApply

# Check code formatting
./gradlew spotlessCheck

# Run detekt static analysis (config: config/detekt/detekt.yml)
./gradlew detekt

# Run Android lint
./gradlew lint
./gradlew :source:api:lintDebug
```

### Documentation

```bash
# Generate Dokka API documentation (outputs to docs/)
./gradlew dokkaGenerate
./gradlew dokkaGenerateHtml
```

### Running Samples

```bash
# Build and install sample apps
./gradlew :samples:quickstart:installDebug
./gradlew :samples:custom-flows:installDebug
./gradlew :samples:linear-clone:installDebug
./gradlew :samples:prebuilt-ui:installDebug
```

**Note:** Before running samples, update Clerk publishable keys in `gradle.properties`:
- `QUICKSTART_CLERK_PUBLISHABLE_KEY`
- `CUSTOM_FLOWS_CLERK_PUBLISHABLE_KEY`
- `LINEAR_CLONE_CLERK_PUBLISHABLE_KEY`
- `PREBUILT_UI_CLERK_PUBLISHABLE_KEY`

### Publishing (Maintainers Only)

```bash
# Publish to Maven Central
./gradlew :source:api:publishToMavenCentral
./gradlew :source:ui:publishToMavenCentral
```

## Architecture Overview

### Module Structure

```
clerk-android/
├── source/
│   ├── api/              # Core authentication logic (clerk-android-api)
│   ├── ui/               # Jetpack Compose UI components (clerk-android-ui)
│   └── telemetry/        # Kotlin Multiplatform telemetry
├── samples/              # Example implementations
│   ├── quickstart/       # Basic integration
│   ├── custom-flows/     # Advanced custom auth flows
│   ├── linear-clone/     # OAuth/Passkey/Email flows with Compose navigation
│   └── prebuilt-ui/      # Prebuilt UI component examples
├── config/               # Repo-wide lint and static-analysis settings
└── workbench/            # Internal development tools
```

### API Module (`source/api`)

**Core Architecture:**
- Retrofit 2 for HTTP with OkHttp middleware pipeline
- Kotlinx Serialization with snake_case naming
- Result-based error handling via `ClerkResult<T, E>` sealed interface (no exceptions)
- Reactive state management with Kotlin StateFlows

**Key Components:**

1. **`Clerk` (singleton object)** - Main SDK entry point at `com.clerk.api.Clerk`
   - Initialize: `Clerk.initialize(publishableKey, context)`
   - Reactive state: `Clerk.sessionFlow`, `Clerk.userFlow`, `Clerk.isInitialized`
   - Lazy initialization with error recovery

2. **Authentication Flow Classes:**
   - `SignIn` - Sign-in state machine with factor verification
   - `SignUp` - Sign-up state machine with field collection
   - `Session` - Active session management with JWT token handling
   - `User` - User profile and account management

3. **Network Layer:**
   - `ClerkApi` - Retrofit service configuration
   - `network/api/*` - Service interfaces (ClientApi, SignInApi, SignUpApi, SessionApi, UserApi, etc.)
   - `network/middleware/*` - Request/response interceptors (versioning, client syncing, device attestation)
   - `network/model/*` - Serializable data models (Client, User, Session, Factor, Verification, etc.)

4. **Authentication Features:**
   - `passkeys/` - WebAuthn/Passkey via Google Credential Manager
   - `sso/` - OAuth/SSO providers (Google One Tap, enterprise SSO)
   - `organizations/` - Multi-tenant organization support
   - `session/` - JWT token caching and refresh

**Result Handling Pattern:**
```kotlin
when (val result = signIn.attemptFirstFactor(...)) {
    is ClerkResult.Success -> // Handle success
    is ClerkResult.Failure -> // Handle error
}
```

### UI Module (`source/ui`)

**Core Architecture:**
- Jetpack Compose with Material Design 3
- MVVM pattern with ViewModels per screen
- CompositionLocal for dependency injection (`LocalAuthState`, `LocalTelemetryCollector`)
- Custom navigation using NavBackStack (androidx.navigation3)
- Paparazzi for snapshot testing

**Key Components:**

1. **Authentication Screens:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clerk/clerk-android](https://github.com/clerk/clerk-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
