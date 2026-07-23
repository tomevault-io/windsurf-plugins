---
trigger: always_on
description: **SWR for Compose** is a Kotlin Multiplatform library that ports [React SWR](https://swr.vercel.app/) to Jetpack Compose / Compose Multiplatform. It implements the "stale-while-revalidate" HTTP cache invalidation strategy — serve cached data immediately while revalidating in the background.
---

# Project Overview

**SWR for Compose** is a Kotlin Multiplatform library that ports [React SWR](https://swr.vercel.app/) to Jetpack Compose / Compose Multiplatform. It implements the "stale-while-revalidate" HTTP cache invalidation strategy — serve cached data immediately while revalidating in the background.

Supported platforms: Android, iOS, JVM (Desktop), Web (JS/WASM).

# Architecture

## Three-Layer Module Design

The library follows a strict layered architecture. Each layer depends only on the one below it:

```
swr-compose   (Compose UI integration — rememberSWR, SWRState, CompositionLocals)
    ↓
swr-runtime   (Lifecycle & network awareness — SWR, SWRMutation, SWRConfig)
    ↓
swr-store     (Pure data caching — SWRStore, DataSelector, SWRStoreState)
```

- **swr-store**: Stateless cache layer. Manages data storage, state transitions (`Loading`/`Completed`/`Error`), and persistence. No lifecycle or platform dependencies.
- **swr-runtime**: Orchestration layer. Adds lifecycle monitoring, network status awareness, automatic revalidation (on focus, reconnect, interval), error retry, and deduplication. Also provides `SWRSubscription` for real-time data sources. Contains platform-specific `expect/actual` implementations for `NetworkMonitor`.
- **swr-compose**: Thin Compose wrapper. Converts runtime flows into Compose state via `rememberSWR` family of functions. Provides `CompositionLocal` for global configuration (`LocalSWRConfig`, `LocalSWRCacheOwner`).

Other modules:
- **exampleApp**: Multi-platform example app demonstrating all features.
- **build-logic**: Gradle convention plugin for Maven Central publishing (Dokka, signing, POM metadata).

## Core Concepts (mirroring React SWR)

| Concept | Description |
|---------|-------------|
| `rememberSWR` | Basic data fetching with automatic revalidation |
| `rememberSWRMutation` | Mutations with optimistic updates and rollback |
| `rememberSWRInfinite` | Paginated/infinite list fetching |
| `rememberSWRImmutable` | Fetch once, never revalidate |
| `rememberSWRPreload` | Prefetch data without lifecycle binding |
| `rememberSWRSubscription` | Subscribe to real-time data sources (WebSocket, SSE, etc.) with SWR cache integration |
| `SWRConfig` | Global/scoped configuration via CompositionLocal |

## Key Design Patterns

- **Sealed state classes**: `SWRStoreState<T>` and `SWRState<DATA>` use sealed class hierarchies (`Loading`, `Completed`, `Error`) for exhaustive state handling.
- **Lambda DSL configuration**: All `rememberSWR*` functions accept a trailing lambda `SWRConfig<KEY, DATA>.() -> Unit` for inline config.
- **Flow-based reactivity**: All internal state is `Flow`/`StateFlow`. Compose layer collects these as Compose state.
- **`expect`/`actual` for platform code**: Network monitoring has platform-specific implementations (ConnectivityManager on Android, NWPathMonitor on iOS, NetworkInterface polling on JVM).
- **`@Immutable` state holders**: All Compose state classes are annotated `@Immutable` for recomposition optimization.
- **`internal` visibility for implementation**: Implementation details (`DataSelector`, `SWRInternal`, `SWRValidate`, `DataState`) are `internal`. Public API surface is minimal.
- **Subscription lifecycle**: `rememberSWRSubscription` uses `SideEffect` (not `DisposableEffect`) to cancel the previous subscription only on key change — NOT when the composable leaves composition. This allows an external `scope` (e.g. `viewModelScope`) to keep the subscription alive across screen transitions. The `subscribe` lambda returns a `Flow<DATA>`; the library collects it internally and writes emitted values to the shared `SWRStore` cache.

## React SWR Reference

This library mirrors [React SWR's API](https://swr.vercel.app/docs/api). When adding features or fixing bugs, consult the React SWR documentation to ensure behavioral consistency. Key parallels:
- `useSWR` → `rememberSWR`
- `useSWRMutation` → `rememberSWRMutation`
- `useSWRInfinite` → `rememberSWRInfinite`
- `useSWRSubscription` → `rememberSWRSubscription`
- `SWRConfig` provider → `LocalSWRConfig` CompositionLocal
- Config options (`revalidateOnFocus`, `dedupingInterval`, `errorRetryCount`, etc.) match React SWR naming.

# Development Commands

## Build
```bash
./gradlew swr-compose:build swr-runtime:build swr-store:build     # Build all library modules
```

## Test
```bash
# Fast: JVM tests only (recommended for development iteration)
./gradlew swr-compose:jvmTest swr-runtime:jvmTest swr-store:jvmTest

# Full: All platform tests (requires Xcode and Android SDK)
./gradlew swr-compose:allTests swr-runtime:allTests swr-store:allTests
```

Most tests live in `swr-runtime` module under `commonTest`. The `swr-store` module has some tests. The `swr-compose` module has minimal tests due to Compose testing complexity.

## Example App
```bash
./gradlew :desktopApp:run                # Desktop (JVM)
./gradlew :webApp:wasmJsBrowserRun       # Web (WASM)
./gradlew :androidApp:installDebug       # Android
```

# Coding Conventions

## Explicit API Mode

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KazaKago/swr-compose](https://github.com/KazaKago/swr-compose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
