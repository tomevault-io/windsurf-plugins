---
trigger: always_on
description: This is a **Kotlin-first, multiplatform SDK for Firebase**. It wraps the official Firebase platform SDKs (Android, iOS, JS, JVM) behind a unified Kotlin common API, enabling Firebase to be used directly from shared Kotlin Multiplatform (KMP) source sets targeting **Android**, **iOS**, **Desktop (JVM)**, and **Web (JS)**.
---

# GitHub Copilot Instructions — firebase-kotlin-sdk

## Project Overview

This is a **Kotlin-first, multiplatform SDK for Firebase**. It wraps the official Firebase platform SDKs (Android, iOS, JS, JVM) behind a unified Kotlin common API, enabling Firebase to be used directly from shared Kotlin Multiplatform (KMP) source sets targeting **Android**, **iOS**, **Desktop (JVM)**, and **Web (JS)**.

All modules are published under the `dev.gitlive` group ID (e.g. `dev.gitlive:firebase-firestore`).

---

## Architecture

### Module Structure

Each Firebase product is a separate Gradle module (e.g. `firebase-auth`, `firebase-firestore`, `firebase-database`). Inside each module the source is split by KMP targets:

```
src/
  commonMain/   ← shared public API (Kotlin)
  androidMain/  ← wraps Firebase Android SDK (and is also used as the physical dir for `jvmMain` in some modules, e.g. firebase-firestore)
  appleMain/    ← shared Apple targets (iOS/tvOS/macOS) wrapping Firebase iOS SDK via Kotlin/Native
  jsMain/       ← wraps Firebase JS SDK
  jvmMain/      ← JVM desktop/server target (may map to src/androidMain/kotlin in some modules)
  commonTest/   ← shared tests
  androidTest/
  appleTest/
  jsTest/
  jvmTest/
```

The `commonMain` source set defines the **public API**. Platform-specific source sets contain the `actual` implementations that delegate to the respective native SDK.

### Shared modules

- `firebase-app` — `FirebaseApp` and `Firebase` object
- `firebase-common` — shared types, serialization helpers, `FirebaseEncoder`/`FirebaseDecoder`
- `firebase-common-internal` — internal utilities not part of the public API

---

## Kotlin-First Design Principles

These principles **must** be followed in all new and modified code.

### 1. Suspend functions instead of callbacks or Tasks

Async operations that return a single value use `suspend fun`. Never use callbacks, `Task`, `Promise`, or listener patterns in `commonMain`.

```kotlin
// ✅ Correct
suspend fun signInWithEmailAndPassword(email: String, password: String): AuthResult

// ❌ Wrong
fun signInWithEmailAndPassword(email: String, password: String, callback: (AuthResult) -> Unit)
```

### 2. `Flow` instead of listeners

Streams of values use `kotlinx.coroutines.flow.Flow`. The flow should be cold — a new listener is registered on collection and removed on cancellation/completion.

```kotlin
// ✅ Correct
val snapshots: Flow<DocumentSnapshot>

// ❌ Wrong
fun addSnapshotListener(listener: (DocumentSnapshot) -> Unit): ListenerRegistration
```

### 3. Default arguments instead of the Builder pattern

Prefer Kotlin default arguments over builder classes. When the upstream Android SDK uses a Builder, provide a Kotlin-idiomatic overload with default arguments **in addition to** accepting the built object (for API compatibility).

```kotlin
// ✅ Correct
suspend fun updateProfile(displayName: String? = null, photoURL: String? = null)

// ❌ Avoid as the sole API
suspend fun updateProfile(request: UserProfileChangeRequest)
```

### 4. Infix notation for query operators

Firestore and Database query operators use infix functions inside a `where { }` builder lambda.

```kotlin
citiesRef.where { "state" equalTo "CA" }
citiesRef.where { "regions" contains "west_coast" }
citiesRef.where {
    all(
        "state" equalTo "CA",
        any("capital" equalTo true, "population" greaterThanOrEqualTo 1_000_000)
    )
}
```

### 5. Operator overloading where natural

Use operator overloading where semantics are obvious (e.g. callable HTTP Functions via `invoke`).

---

## Serialization

The SDK uses **`kotlinx.serialization`** throughout. Never use platform-specific serialization mechanisms in `commonMain`.

- Custom classes passed to/from Firebase must be annotated with `@Serializable`.
- Always accept an explicit `SerializationStrategy`/`DeserializationStrategy` parameter alongside a reified/inferred overload.
- `encodeDefaults` defaults to `true`; allow it to be overridden via a `buildSettings` lambda.
- Support `serializersModule` for contextual and polymorphic serialization.
- Use `@FirebaseClassDiscriminator` (defined in `firebase-common`) on sealed classes to control the type discriminator field name.
- Special sentinel values (`ServerValue.TIMESTAMP`, `Timestamp.ServerTimestamp`, `FieldValue.serverTimestamp`) must remain serializable.
- For Firestore update operations, provide an `updateFields` builder that allows per-field serializer overrides.

---

## API Compatibility Goal

The target is **near binary compatibility** with the [Firebase Android SDK Kotlin API](https://firebase.google.com/docs/reference/kotlin/packages):

- Match class names, function names, and parameter names from the Android SDK.
- Package imports should be the **only change** needed when porting Android code: `com.google.firebase` → `dev.gitlive.firebase`.
- When an Android SDK API is Java-first (uses builders, callbacks, etc.), provide **both** the Android-compatible form *and* a Kotlin-idiomatic overload.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GitLiveApp/firebase-kotlin-sdk](https://github.com/GitLiveApp/firebase-kotlin-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
