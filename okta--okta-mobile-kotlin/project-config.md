---
trigger: always_on
description: Kotlin Multiplatform SDK for Okta authentication (Android, JVM). Code guidelines for anyone (human or AI) writing or reviewing code in this repository.
---

# AGENTS.md — okta-mobile-kotlin

Kotlin Multiplatform SDK for Okta authentication (Android, JVM). Code guidelines for anyone (human or AI) writing or reviewing code in this repository.

## Architecture & Design Principles

- **Immutability**: any class property that is public or reachable through a public method must be immutable.
- **Model separation**: keep three distinct kinds of models — a serializable *data model* for persistent storage, a non-serializable *domain model* for business logic, and a serializable *network model* used only for (de)serializing requests/responses.
- **Singletons are discouraged**, except for narrow cases such as a shared database instance.
- **Builder pattern** for any class with more than 5 configurable parameters (e.g. `OAuth2ClientBuilder`, `DirectAuthenticationFlowBuilder`).
- **Functional programming** patterns are preferred where they keep code clear; use imperative style when it is more concise.
- **`lateinit`.** is discouraged, except for cases such as tests.
- **Single source of truth**: state must have exactly one authoritative owner.
- **Depend on abstractions, not concretions**: even without a DI framework, wire dependencies via constructor parameters and code against interfaces rather than reaching for concrete singletons inside business logic.

### Module Dependency Hierarchy

```
auth-foundation           (KMP: Android+JVM) — root dependency, no internal SDK deps
├── oauth2                (KMP: Android+JVM) — AuthorizationCodeFlow, PKCE, token exchange
├── web-authentication-ui (Android-only) — browser redirect UI; depends on oauth2
├── legacy-token-migration (Android-only) — migrate from legacy Okta SDKs
├── okta-idx-kotlin       (Android-only) — IDX interaction code flow
│   └── native-authentication (Android-only)
├── okta-direct-auth      (KMP: Android+JVM) — direct auth flow
│   └── okta-direct-auth-shared (KMP: Android+JVM, Compose Multiplatform UI)
│       — also depends on oauth2 (commonMain) and web-authentication-ui (androidMain)
└── bom                   — Maven BOM (java-platform); constrains auth-foundation,
                             web-authentication-ui, oauth2, legacy-token-migration,
                             okta-idx-kotlin, okta-direct-auth (not native-authentication
                             or okta-direct-auth-shared)
```

Note: `okta-direct-auth` has no `js()`/`wasmJs()` target — despite the module name/README implying JS+WASM support, it is currently Android+JVM only.

Sample apps: `app`, `okta-direct-auth-android-app`, `okta-direct-auth-desktop-app`, `session-token-sample`, `legacy-token-migration-sample`, `dynamic-app`

### KMP Source Sets

Multiplatform modules (`auth-foundation`, `okta-direct-auth`, `okta-direct-auth-shared`) use:
`commonMain` / `androidMain` / `jvmMain` with corresponding test sets (`commonTest`, `androidHostTest`, `androidDeviceTest`, `jvmTest`).

Android-only libraries use standard `src/main/` and `src/test/` layout.

### JVM wrapper parity with Kotlin

`jvm` sub-packages provide Java-idiomatic builders/flows (method-chaining setters, `AuthFoundationResult`/`DirectAuthResult` instead of `kotlin.Result`, no suspend fun in the public surface) that delegate to a KMP commonMain Kotlin API — e.g. auth-foundation's `OAuth2ClientBuilder`/`TokenCredentialManager`, oauth2's flow wrappers (`AuthorizationCodeFlow`, `DeviceAuthorizationFlow`, etc.), okta-direct-auth's `DirectAuthenticationFlowBuilder`.

- These jvm wrappers MUST stay at full parity with their commonMain counterpart: every public property, setter, and method on the commonMain API needs a corresponding Java-friendly equivalent on the jvm wrapper, unless there's a documented reason (e.g. a suspend-only API with no Java-callable equivalent, or an intentionally Java-convenience-only method).
- When editing a commonMain builder or flow (adding/renaming/removing a public property, setter, or method), update its jvm wrapper in the same change — don't defer it. Check `git log --oneline -- <commonMain file>` vs the jvm file's history if unsure whether drift already exists.
- Found via real drift in `OAuth2ClientBuilder`, `TokenCredentialManager`, and `DirectAuthenticationFlowBuilder`.
- Keep the public surface narrow: prefer `internal`/`private` over `public` unless a member is genuinely meant for external callers. Every public member on a commonMain API is a mandatory parity obligation on its jvm wrapper, so a smaller public surface means a smaller, cheaper-to-maintain wrapper.

### Key Patterns

- **OAuth2Client** (`auth-foundation`) — low-level Okta authorization server client; uses `CoalescingOrchestrator` for request deduplication
- **Credential / CredentialDataSource** — token management with `RoomTokenStorage` (SQLCipher-encrypted Room DB) and biometric support
- **Flow classes** — each auth method is a flow: `AuthorizationCodeFlow`, `ResourceOwnerFlow`, `DeviceAuthorizationFlow`, `SessionTokenFlow`, `TokenExchangeFlow`, `RedirectEndSessionFlow` (oauth2),
  `InteractionCodeFlow` (idx), `DirectAuthenticationFlow` (direct-auth). Platform-specific flow implementations must honor the same contract — no platform-only exceptions, ordering, or side

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okta/okta-mobile-kotlin](https://github.com/okta/okta-mobile-kotlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
