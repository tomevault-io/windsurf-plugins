---
trigger: always_on
description: Guidance for AI agents and contributors working in this repository.
---

# CLAUDE.md

Guidance for AI agents and contributors working in this repository.

## What this is

KMPAuth — a Kotlin Multiplatform authentication library (Google, Apple, GitHub, Facebook, Microsoft, email/password, phone and anonymous sign-in, with optional Firebase integration and a pluggable backend abstraction). Published to Maven Central as `io.github.mirzemehdi:kmpauth-*`. Version lives in `gradle.properties` (`kmpAuthVersion`).

## Module map

Layout: identity **providers** (credential sources) live under `providers/`; session **backends** under `backends/` (Firebase and Supabase). Directory grouping only — artifact ids are the project names. Legacy compatibility shims live under `deprecated/`.

| Module (gradle path) | Purpose | Depends on |
|---|---|---|
| `:kmpauth-core` | Base infrastructure: logging, `SignInState`/`LaunchingSignInState`/`UnsupportedSignInState`, `runCatchingCancellable`, the `KMPAuth` client facade, the `com.mmk.kmpauth.core.auth` backend abstraction (`AuthProviderBackend`, `KMPAuthBackend`, `AuthCredential`, `KMPAuthUser`, `EmailActionCodeSettings`) and the backend-generic auth states (`rememberEmailAuthState`, `rememberAnonymousAuthState`, `rememberOAuthState`/`rememberGithubAuthState`/`rememberMicrosoftAuthState` (browser OAuth via `signIn(OAuthWebFlow)`), `rememberPhoneAuthState` + `PhoneAuthState`/`PhoneVerificationUi`; Android supplies the Activity via the lifecycle-tracked `AndroidActivityTracker` — core has no compose.ui dependency, so no `LocalContext`) | — |
| `:providers:kmpauth-google` | Google Sign-In (Credential Manager on Android, GoogleSignIn SDK on iOS, OAuth loopback on JVM): `rememberGoogleSignInState` (credential only) + `rememberGoogleAuthState` (session via the registered backend) | core |
| `:providers:kmpauth-facebook` | Facebook Login via Facebook SDK: `rememberFacebookSignInState` (credential only) + `rememberFacebookAuthState` (session via the registered backend) | core |
| `:providers:kmpauth-apple` | `rememberAppleAuthState` (backend-generic session: native AuthenticationServices credential -> `signIn(IdToken)` incl. first-authorization displayName on iOS; `signIn(OAuthWebFlow("apple.com"))` elsewhere) + `rememberAppleSignInState` (credential only). Native flow is iOS-only — other targets report a failed `Result` (`UnsupportedSignInState`), since Apple's web flow needs a server-side client-secret exchange. Also supplies the iOS credential acquisition reused by kmpauth-firebase (`performAppleSignIn`, `@KMPAuthInternalApi`) | core |
| `:backends:firebase:kmpauth-firebase` | `FirebaseAuthBackend` (default backend) with per-platform engines: GitLive firebase-auth on android/ios/js, **Firebase Auth REST API on JVM and wasm** (`FirebaseRestAuthEngine` in the shared `restEngineMain` source set + kotlinx-serialization-json runtime; transports: JDK HttpClient on jvm, fetch on wasm; session persisted via `FirebaseSessionStorage` expect/actual - `~/.kmpauth/` file on jvm, localStorage on wasm, restored sessions marked stale and refreshed on first use - GitLive's java-sdk has no auth #204, and GitLive has no wasm target #179). Engine web-flow/phone pieces: `gitLiveOAuthWebFlowSignIn` (android `startActivityForSignInWithProvider` with the Activity from core's `AndroidActivityTracker`, ios FIR provider flow; js/jvm actuals throw - jvm uses the REST engine's DesktopWebAuthFlow) and the GitLive `PhoneVerificationProvider` factories. All OAuth/Apple/phone composables now live in core/kmpauth-apple, NOT here. Its iOS Apple flow delegates to `:providers:kmpauth-apple` | core, apple |
| `:deprecated:kmpauth-firebase-google` | Deprecated 2.x `GoogleButtonUiContainerFirebase` only (the auth state moved to `kmpauth-google`); removal in 4.0 | firebase, google |
| `:deprecated:kmpauth-firebase-facebook` | Deprecated 2.x `FacebookButtonUiContainerFirebase` only (the auth state moved to `kmpauth-facebook`); removal in 4.0 | firebase, facebook |
| `:backends:supabase:kmpauth-supabase` | `SupabaseAuthBackend` over community supabase-kt (`auth-kt`, all targets incl. wasm — no nonWasmMain split needed). Explicit registration only: `KMPAuth.initialize { supabase(...) }` (a Supabase client needs url+key, so no ServiceLoader/eager auto-registration). No composables — the backend-generic states in core/google/facebook already serve it. supabase-kt is Ktor-based; consumers add their platform's Ktor engine (this is supabase-kt's requirement, not a KMPAuth HTTP dependency) | core |
| `:kmpauth-uihelper` | Pre-styled Compose sign-in buttons (Google/Apple/Facebook) | core |
| `sampleApp/shared` + `androidApp`/`desktopApp`/`webApp`/`iosApp` | Demo: shared UI module + per-platform entry points (webApp builds both js and wasm variants — the wasm one is the smoke test for the wasm-callable Firebase API) | all |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mirzemehdi/KMPAuth](https://github.com/mirzemehdi/KMPAuth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
