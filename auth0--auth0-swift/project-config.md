---
trigger: always_on
description: This document provides context and guidelines for AI coding assistants working with the Auth0.swift codebase.
---

# AI Agent Guidelines for Auth0.swift

This document provides context and guidelines for AI coding assistants working with the Auth0.swift codebase.

## Project Overview

**Auth0.swift** is the official Auth0 SDK for Apple platforms — providing authentication, authorization, and credential management for iOS, macOS, tvOS, watchOS, and visionOS apps.

- **Language:** Swift 5.0+ (Package.swift uses Swift 6.0 tools)
- **Tech Stack:** Apple platforms, Xcode 16.x, SPM + CocoaPods + Carthage, URLSession, Combine, CryptoKit
- **Package Manager:** Swift Package Manager (primary), CocoaPods, Carthage (development deps)
- **Minimum Platform Versions:** iOS 14.0, macOS 11.0, tvOS 14.0, watchOS 7.0, visionOS 1.0

---

## Commands

> Copy-paste ready. These are the exact commands used in CI.

```bash
# Run all tests via Swift Package Manager (fastest local option)
swift test

# Run tests for a specific Xcode scheme (used in CI)
xcodebuild test -project Auth0.xcodeproj -scheme Auth0.iOS -destination 'platform=iOS Simulator,name=iPhone 16'
xcodebuild test -project Auth0.xcodeproj -scheme Auth0.macOS -destination 'platform=macOS'
xcodebuild test -project Auth0.xcodeproj -scheme Auth0.tvOS -destination 'platform=tvOS Simulator,name=Apple TV'

# Lint (must pass before merging)
swiftlint lint --reporter github-actions-logging

# Lint with auto-fix
swiftlint lint --fix

# Bootstrap Carthage dependencies (required for Xcode project development)
carthage bootstrap --use-xcframeworks

# Bootstrap Carthage for a specific platform
carthage bootstrap --platform iOS --use-xcframeworks --no-use-binaries --cache-builds

# Coverage report (iOS only, run after tests)
bundle exec slather coverage -x --scheme Auth0.iOS Auth0.xcodeproj

# Validate CocoaPods spec
bundle exec pod lib lint --allow-warnings --fail-fast

# Resolve SPM dependencies
xcodebuild -resolvePackageDependencies -skipPackageUpdates -onlyUsePackageVersionsFromResolvedFile

# Generate API documentation (DocC via Fastlane)
bundle exec fastlane build_docs
```

---

## Testing

- **Framework:** Quick 7.0+ (BDD) + Nimble 13.0+ (assertions)
- **Test Location:** `Auth0Tests/`
- **Coverage Tool:** Slather + Codecov (iOS scheme only in CI)
- **Coverage Threshold:** Tracked via Codecov; target >80%

### Running Tests

```bash
# Run all unit tests via SPM (quickest)
swift test

# Run a specific test spec via xcodebuild
xcodebuild test -project Auth0.xcodeproj \
  -scheme Auth0.iOS \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:Auth0Tests/CredentialsManagerSpec
```

### Testing Conventions

- Every spec file is a `QuickSpec` subclass named `<Subject>Spec` (e.g., `CredentialsManagerSpec`).
- Behavior is organized with nested `describe` / `context` / `it` blocks.
- `it` descriptions use present tense, declarative style: `"should return credentials when valid"`.
- `beforeEach` / `afterEach` handle setup and teardown.
- `StubURLProtocol` intercepts all network calls — never make real network requests in tests.
- `NetworkStub.clearStubs()` must be called in every `afterEach`.
- Test constants use `UPPER_CAMEL_CASE` names (e.g., `AccessToken`, `ClientId`, `Domain`).
- Combine publishers are tested with Nimble async matchers or `waitUntil`.
- Platform-specific tests are gated with `#if WEB_AUTH_PLATFORM` and `#if PASSKEYS_PLATFORM`.

### Mocking & Test Utilities

- Network: `StubURLProtocol` + `NetworkStub` (register/clear stubs per test)
- Keychain: `SimpleKeychain` is used directly; tests clean up Keychain state in `afterEach`
- Platform guards: tests mirror the same `#if WEB_AUTH_PLATFORM` / `#if PASSKEYS_PLATFORM` flags as source

---

## Project Structure

```
Auth0.swift/
├── Auth0/                        # Library source (85 Swift files)
│   ├── Auth0.swift               # Public result type aliases & top-level factory functions
│   ├── Authentication.swift      # Authentication protocol (OAuth2 / OIDC)
│   ├── Auth0Authentication.swift # Concrete Authentication implementation
│   ├── AuthenticationError.swift # Authentication API error type
│   ├── CredentialsManager.swift  # Thread-safe Keychain credential storage & renewal
│   ├── CredentialsManagerError.swift
│   ├── Credentials.swift         # User credentials model
│   ├── WebAuth.swift             # Web Auth protocol (Universal Login)
│   ├── Auth0WebAuth.swift        # Concrete WebAuth implementation
│   ├── WebAuthError.swift
│   ├── Version.swift             # Single source of truth for SDK version string
│   ├── DPoP/                     # DPoP (Demonstration of Proof-of-Possession) support
│   │   ├── DPoP.swift
│   │   └── DPoPError.swift
│   ├── MFA/                      # Multi-factor authentication
│   │   ├── MFAClient.swift
│   │   ├── Auth0MFAClient.swift
│   │   └── MFAErrors.swift
│   ├── MyAccount/                # My Account API
│   │   ├── MyAccount.swift
│   │   ├── MyAccountError.swift
│   │   └── AuthenticationMethods/
│   └── Utils/                    # Internal utilities
├── Auth0Tests/                   # Test specs (56 files, mirrors Auth0/ structure)
│   ├── Auth0Spec.swift
│   ├── AuthenticationSpec.swift
│   ├── CredentialsManagerSpec.swift
│   ├── DPoP/
│   ├── MFA/
│   └── MyAccount/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [auth0/Auth0.swift](https://github.com/auth0/Auth0.swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
