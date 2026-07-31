---
trigger: always_on
description: Guidance for AI coding assistants working on the Auth0.Android SDK.
---

# CLAUDE.md — Auth0.Android SDK

Guidance for AI coding assistants working on the Auth0.Android SDK.

## Project Overview

**Auth0.Android** is a native Android SDK for authentication and authorization with Auth0. It provides:

- Web authentication via Universal Login and browser-based OAuth flows (WebAuthProvider)
- Direct authentication APIs (database, passwordless, OTP, token operations)
- Credential persistence and automatic token renewal via CredentialsManager
- Secure credential storage using Android Keystore with biometric integration
- Kotlin-first APIs with full Java interoperability and coroutine support
- MFA (Multi-Factor Authentication) support with flexible factor enrollment and verification
- My Account API for self-service authentication method management
- DPoP (Demonstration of Proof-of-Possession) for sender-constrained tokens
- Passkey support for passwordless FIDO2 authentication

**Technology Stack:**
- **Language:** Kotlin (with Java interoperability)
- **Min SDK:** API 26, Compile/Target: API 36
- **Java source/target:** 17
- **Kotlin JVM target:** 17
- **Build System:** Gradle 8.10.1 with Android Gradle Plugin 8.10.1
- **Kotlin:** 2.0.21 with explicit API mode (strict)
- **Package Manager:** Gradle + Maven Central
- **Current Version:** 4.0.0

---

## Commands

```bash
# Run unit tests and generate coverage reports (matching CI pipeline)
./gradlew testReleaseUnitTest jacocoTestReleaseUnitTestReport lintRelease --continue --console=plain

# Run all lint checks
./gradlew lint

# Build the SDK library
./gradlew auth0:assemble

# Build sample/demo app
./gradlew sample:assembleDebug

# Build release AAR for publishing
./gradlew auth0:assembleRelease

# Build and run all tests with output
./gradlew test --info

# Clean build artifacts
./gradlew clean

# Check Kotlin API surface for explicit API mode (strict) violations
./gradlew auth0:compileReleaseKotlin
```

---

## Testing

**Framework & Tools:**
- JUnit 4 (4.13.2)
- Robolectric 4.15.1 (Android framework mocking)
- Mockito Core 5.14.0 + Mockito Kotlin 5.4.0
- MockWebServer (4.12.0) + OkHttp TLS helpers for HTTP mocking
- Hamcrest matchers
- Awaitility (async verification)
- Kotlin Coroutines Test
- Espresso Intents (for browser interaction tests)

**Test Locations:**
```text
auth0/src/test/
├── java/com/auth0/android/
│   ├── authentication/          # Auth API client tests
│   ├── provider/                # WebAuthProvider/browser tests
│   ├── authentication/storage/  # CredentialsManager tests
│   ├── authentication/mfa/      # MFA API tests
│   ├── dpop/                    # DPoP tests
│   ├── myaccount/               # My Account API tests
│   └── result/                  # Response parsing tests
```

**Coverage Requirements:**
- Minimum target: 80% for patch changes
- Ignored from coverage: `CryptoUtil.java` (Android framework-dependent encryption)
- Both success and failure/error paths required for new behavior
- Async code must use Awaitility or coroutine test helpers to control execution

**Testing Patterns:**
- Use `@RunWith(RobolectricTestRunner.class)` for Android component tests
- Mock HTTP responses with MockWebServer; validate URLs/headers with matchers
- Mock `Context` with Mockito for framework interactions
- Use `coroutineScope.advanceUntilIdle()` or Awaitility for async/callback verification
- Avoid `PowerMock`; prefer constructor/method injection with mocks (project is removing PowerMock)

---

## Project Structure

```text
Auth0.Android/
├── auth0/                                      # Main SDK library module
│   ├── src/main/java/com/auth0/android/
│   │   ├── Auth0.kt                            # SDK config entry point (client ID, domain, networking)
│   │   ├── Auth0Exception.kt                   # Base exception type
│   │   ├── authentication/
│   │   │   ├── AuthenticationAPIClient.kt      # Direct auth API (login, signup, MFA, passwordless)
│   │   │   ├── AuthenticationException.kt      # Auth API error type
│   │   │   ├── storage/
│   │   │   │   ├── CredentialsManager.kt       # Token caching and refresh (callback + coroutine APIs)
│   │   │   │   ├── SecureCredentialsManager.kt # Encrypted storage + biometric integration
│   │   │   │   └── SharedPreferencesStorage.kt # Persistence adapter
│   │   │   ├── mfa/
│   │   │   │   ├── MfaApiClient.kt             # MFA enrollment, challenge, verify
│   │   │   │   └── MfaException.kt
│   │   │   └── passwordless/
│   │   │       └── PasswordlessClient.kt       # Passwordless flows
│   │   ├── provider/
│   │   │   ├── WebAuthProvider.kt              # Browser login/logout builder + coroutines
│   │   │   ├── AuthenticationActivity.kt       # Redirect callback handling
│   │   │   ├── OAuthManager.kt                 # Internal flow orchestration
│   │   │   └── [browser option helpers]
│   │   ├── myaccount/
│   │   │   ├── MyAccountAPIClient.kt           # My Account API (authentication method management)
│   │   │   └── MyAccountException.kt
│   │   ├── dpop/
│   │   │   ├── DPoP.kt                         # DPoP proof generation and nonce handling
│   │   │   ├── DPoPKeyStore.kt                 # Key pair management
│   │   │   ├── DPoPException.kt
│   │   │   └── SenderConstraining.kt           # DPoP integration interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [auth0/Auth0.Android](https://github.com/auth0/Auth0.Android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
