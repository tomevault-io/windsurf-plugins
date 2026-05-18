---
trigger: always_on
description: Operational guide for AI coding agents (and new contributors) working in this repository.
---

# AGENTS.md — KtorMonitor

Operational guide for AI coding agents (and new contributors) working in this repository.

> Keep this file up to date when build commands, module layout, conventions or coding standards change.

---

## 1. Project overview

**KtorMonitor** is a Kotlin Multiplatform library that intercepts and visualizes HTTP traffic from
[Ktor Client](https://ktor.io/) and [OkHttp](https://square.github.io/okhttp/), with a Compose
Multiplatform UI for inspecting requests and responses.

- **Targets:** Android, iOS (arm64 / simulator arm64), Desktop JVM (Windows / macOS / Linux), JS (browser), Wasm/JS (browser).
- **Tech stack:** Kotlin **2.3.21**, Compose Multiplatform **1.11.0-beta01**, Ktor **3.4.3**, OkHttp **5.3.2**, http4k **6.15.1.0**, SQLDelight **2.3.2** (async driver, `sql.js` on web), Koin **4.2.1**, Coil **3.4.0**, kotlinx.atomicfu, kotlinx.coroutines, kotlinx.datetime, kotlinx.serialization.
- **Group / artifacts** (group `ro.cosminmihu.ktor`):
  - `ktor-monitor-core`            — shared core (UI + storage)
  - `ktor-monitor-core-no-op`      — ABI-equivalent no-op
  - `ktor-monitor-logging`         — Ktor client plugin
  - `ktor-monitor-logging-no-op`
  - `ktor-monitor-okhttp-interceptor`        — OkHttp interceptor (Android & JVM)
  - `ktor-monitor-okhttp-interceptor-no-op`
  - `ktor-monitor-http4k-filter`             — http4k Filter (Android & JVM)
  - `ktor-monitor-http4k-filter-no-op`
- **Distribution:** Maven Central, signed via `com.vanniktech.maven.publish`, ABI checked via `binary-compatibility-validator`, docs via Dokka + MkDocs Material (`docs/`).

## 2. Module structure

| Path                              | Plugin / target            | Purpose                                                                 |
|-----------------------------------|----------------------------|-------------------------------------------------------------------------|
| `core/library`                    | KMP library (all targets)  | DB (SQLDelight), Compose UI, Koin DI, notification & share managers.    |
| `core/library-no-op`              | KMP library                | API-compatible empty implementation for production builds.              |
| `ktor/library-ktor`               | KMP library                | Ktor client `KtorMonitorLogging` plugin; depends on `core/library`.     |
| `ktor/library-ktor-no-op`         | KMP library                | No-op mirror of `library-ktor`.                                         |
| `okhttp/library-okhttp`           | KMP (Android + JVM)        | `KtorMonitorInterceptor` for OkHttp; depends on `core/library`.         |
| `okhttp/library-okhttp-no-op`     | KMP (Android + JVM)        | No-op mirror of `library-okhttp`.                                       |
| `http4k/library-http4k`           | KMP (Android + JVM)        | `KtorMonitorFilter` for http4k; depends on `core/library`.              |
| `http4k/library-http4k-no-op`     | KMP (Android + JVM)        | No-op mirror of `library-http4k`.                                       |
| `sample/ktor`                     | Compose Multiplatform app  | Demo for Ktor monitor (Android, iOS, JVM, JS, Wasm).                    |
| `sample/okhttp`                   | Compose Multiplatform app  | Demo for OkHttp monitor (Android + JVM).                                |
| `sample/http4k`                   | Compose Multiplatform app  | Demo for http4k monitor (Android + JVM).                                |
| `docs/`                           | MkDocs Material            | Documentation site, including generated Dokka API docs in `docs/api/`.  |

Type-safe project accessors are enabled (`enableFeaturePreview("TYPESAFE_PROJECT_ACCESSORS")`); reference modules as `projects.core.library`, `projects.ktor.libraryKtor`, `projects.http4k.libraryHttp4k`, etc.

### Source-set layout (core)

```
core/library/src/
  commonMain/   — shared Kotlin + Compose UI (ui/, db/, di/, domain/, core/)
  commonMain/sqldelight/  — SQLDelight schema (Call.sq → ro.cosminmihu.ktor.monitor.db.sqldelight)
  commonMain/composeResources/  — Compose Multiplatform resources (drawables, strings)
  androidMain/  — Activity, notification channel, clipboard, share, permission banner
  iosMain/      — UNUserNotificationCenter, UIPasteboard, UIActivityViewController
  jvmMain/      — Swing/AWT clipboard & share, KtorMonitorWindow / Panel / MenuItem
  jsMain/ wasmJsMain/ webMain/ — Web fallbacks; SQLDelight web-worker driver, sql.js webpack plugin
```

## 3. Build, test, run

A wrapper is included; always use `./gradlew`. JVM toolchain is **Java 11** for the libraries and the CI publish job uses **Java 21+**.

| Goal                                  | Command                                                               |
|---------------------------------------|-----------------------------------------------------------------------|
| Full build                            | `./gradlew build`                                                     |
| All checks (lint + tests + apiCheck)  | `./gradlew check`                                                     |
| JVM unit tests (run in CI)            | `./gradlew jvmTest`                                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CosminMihuMDC/KtorMonitor](https://github.com/CosminMihuMDC/KtorMonitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
