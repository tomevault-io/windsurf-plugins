---
trigger: always_on
description: Guidance for Claude Code (and any agent) working in the **LogPose** repository.
---

# CLAUDE.md

Guidance for Claude Code (and any agent) working in the **LogPose** repository.

## What this is

LogPose is an open-source runtime inspector for Android. It reads an app's HTTP traffic out
of **logcat** as clean, structured, per-request transactions — fixing the interleaved lines,
mismatched bodies, and 4 KB truncation you get from ad-hoc network logging.

Since 1.5.0 it's a **framework**, not just an HTTP tool: every timeline event travels in an
`Envelope` with an opaque payload, so an app can log its own kinds (`LogPose.event { }`) and
they render without a plugin release. The same capture is exposed over **MCP**, so a coding
agent can read what the running app actually did — and create mocks to change what it receives
next.

It has **two halves that ship through two different channels** — keep them straight:

| Half | Path | What it is | Distributed via |
| --- | --- | --- | --- |
| **IDE plugin** | repo root (`src/`) | Kotlin + IntelliJ Platform plugin: the tool window that renders transactions | **JetBrains Marketplace** (plugin id **32148**) as a built `.zip` |
| **On-device library** | `logpose-android/` | A drop-in OkHttp `Interceptor` that emits structured transactions to logcat | **JitPack** (a Gradle dependency) |
| **No-op library** | `logpose-android/no-op/` | A pure-JVM stub mirroring the interceptor's public API, for release builds | **JitPack** |

A change to plugin UX goes through the Marketplace zip; a change to capture/emit behavior
goes through the JitPack library. They version independently.

## Build & run

### IDE plugin (repo root)
```bash
./gradlew runIde         # launch a sandbox IDE with the plugin loaded
./gradlew buildPlugin    # → build/distributions/logpose-<version>.zip (Marketplace upload)
./gradlew test           # unit tests (e.g. DuplicateDetectorTest)
./gradlew verifyPlugin   # JetBrains Plugin Verifier against pinned IDEs (2024.1 → 2025.2)
```
- Version is set in `build.gradle.kts` (`version = "..."`).
- `untilBuild` is intentionally open (`provider { null }`) so the plugin loads on newer IDEs.
- The plugin talks to `adb` directly and depends on `bundledPlugin("com.intellij.modules.json")`
  for the Raw-mode editor — it deliberately does **not** depend on the Android plugin, so it
  runs in any JetBrains IDE.

### Signing & publishing the plugin
`signPlugin` / `publishPlugin` read secrets from the environment only (never the repo):
`CERTIFICATE_CHAIN`, `PRIVATE_KEY`, `PRIVATE_KEY_PASSWORD`, `PUBLISH_TOKEN`. Absent vars are
fine for a normal build. For a manual Marketplace upload you can upload the unsigned
`buildPlugin` zip — JetBrains signs it server-side. See `RELEASING.md`.

### On-device library (`logpose-android/`)
It's a separate Gradle build (own `settings.gradle.kts`, own wrapper). JitPack builds it via
`jitpack.yml` on each pushed git tag:
```bash
cd logpose-android && ./gradlew publishToMavenLocal -Pversion=<version>
```
This publishes two artifacts — `logpose-android` (real) and `:no-op` (stub). Current JitPack
coordinates:
```kotlin
debugImplementation("com.github.siddharthjaswal.logpose:logpose-android:<tag>")
releaseImplementation("com.github.siddharthjaswal.logpose:logpose-no-op:<tag>")
```
`okhttp` is `compileOnly` in both (the host app provides it); only the real lib pulls
`kotlinx-serialization`.

## Code map

### Plugin (`src/main/kotlin/io/github/siddharthjaswal/logpose/`)
- `toolwindow/` — `LogPosePanel` (master/detail UI), `TransactionListRenderer` (list rows),
  `LogPoseToolWindowFactory`.
- `ui/KindPresenter.kt` — maps a structured payload (db / worker / config) onto the
  title/badges/sections model a self-describing event supplies for itself, so one row layout and
  one detail view serve every non-HTTP/FCM kind. **Presentation decisions live here, never on
  the wire.**
- `ui/` — `Ui.kt` (`Theme` tokens as `JBColor` light/dark pairs, `TagLabel`, helpers),
  `OverviewPanel`, `TransactionDetailView` (HTTP), `FcmDetailView` (FCM),
  `GenericDetailView` (db / worker / config / app-defined), `JsonTreePanel` (Tree + Raw JSON
  editor),
  `FilterBar` (incl. the `NET`/`FCM`/`DB`/`WORK`/`CONF`/`APP` TYPE toggle), `CurlBuilder`,
  `MutedEndpoints`.
- `logcat/` — `LogcatReader` (tails `adb logcat`, all adb work **off the EDT**),
  `TransactionParser` (reassembles chunked JSON; returns a `LogEvent`, and routes reverse-channel
  `ControlMessage`s — hello / mock-ack — to `onControl`), `Adb` (shared adb resolve + cmd prefix).
- `mock/` — `MocksController`: owns the rule set, persists it per project, and pushes rules to
  the device via `adb shell am broadcast` (**off the EDT**); consumes hello/ack control messages
  to track sync + hit counts. Clears device rules on Stop Capture.
- `mcp/` — the MCP server exposing the live capture to coding agents. `McpTools` (query +
  mock-write logic; **free of HTTP and IntelliJ types, so it's unit-tested**), `McpSessions`
  (per-project token → capture; the token both authenticates and selects the project),
  `LogPoseMcpHandler` (hand-rolled JSON-RPC on the platform's `httpRequestHandler` EP —
  **note that EP resolves under `com.intellij`, not `org.jetbrains`, or it silently never
  loads**). Runs on a Netty IO thread: **never touch Swing there**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [siddharthjaswal/logpose](https://github.com/siddharthjaswal/logpose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
