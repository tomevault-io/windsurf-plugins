---
trigger: always_on
description: Amethyst is a Kotlin Multiplatform (KMP) app for creating and playing Launchpad light-show performances on Desktop (JVM), Android, and iOS. It uses Compose Multiplatform for UI.
---

# Amethyst – Copilot Instructions

Amethyst is a Kotlin Multiplatform (KMP) app for creating and playing Launchpad light-show performances on Desktop (JVM), Android, and iOS. It uses Compose Multiplatform for UI.

---

## Build & Run

All commands use the Gradle wrapper from the repo root.

| Task | Command |
|---|---|
| Run desktop app | `./gradlew :composeApp:run` |
| Build Android debug APK | `./gradlew :composeApp:assembleDebug` |
| Build desktop distributable | `./gradlew :composeApp:packageDistributionForCurrentOS` |
| Run all tests | `./gradlew :composeApp:allTests` |
| Run desktop tests only | `./gradlew :composeApp:desktopTest` |
| Run a single test class | `./gradlew :composeApp:desktopTest --tests "dev.anthonyhfm.amethyst.gem.GemRepositoryTest"` |
| Run Android unit tests | `./gradlew :composeApp:testDebugUnitTest` |

iOS is built via Xcode using the `iosApp/` project. The KMP framework is `ComposeApp` (static).

Desktop entry point: `dev.anthonyhfm.amethyst.MainKt` (`desktopMain/…/main.kt`).

Gradle wrapper version: **9.3.1**. Kotlin: **2.3.0**. Compose Multiplatform: **1.10.0**. Android min/target SDK: 29/36, JVM target: 17.

---

## Architecture Overview

There is one Gradle module (`:composeApp`) with these source sets:

- `commonMain` — all shared logic and Compose UI
- `desktopMain` — JVM-specific implementations
- `androidMain` — Android-specific implementations
- `iosMain` — iOS-specific implementations
- `commonTest` / `androidUnitTest` — tests

All source lives under `composeApp/src/` with the root package `dev.anthonyhfm.amethyst`.

### Top-level packages in `commonMain`

| Package | Purpose |
|---|---|
| `core/engine/heaven` | **Heaven** — singleton rendering engine that converts LED signals into Launchpad output at a configurable FPS using a coroutine-driven scheduler |
| `core/engine/echo` | Audio playback engine (platform-specific decoders via `expect`/`actual`) |
| `core/engine/elements` | `Signal`, `SignalReceiver`, `Chain` — the signal-flow primitives |
| `core/midi` | MIDI device access (`PlatformMidiAccess`, `AmethystMidiManager`); MIDI access is `expect`/`actual` per platform |
| `core/controls` | `UndoManager`, `SelectionManager`, `ShortcutManager`, clipboard, keyboard modifier state |
| `core/util` | `Platform` (sealed interface), `UUID`, `Zip`, `Version`, `StopWatch`, `DeviceCapabilities` — all `expect`/`actual` |
| `devices/` | Every effect in the chain is a `GenericChainDevice<State>` or `LEDChainDevice<State>` subclass; effects live under `devices/effects/`, audio under `devices/audio/` |
| `gem/` | Gem system — reusable visual node-graph programs; see below |
| `workspace/` | Active session state (`WorkspaceRepository` singleton), chain viewport, workspace modes |
| `timeline/` | Timeline/automation system with tracks, lanes, and automation evaluators |
| `ui/` | Shared Compose components, theme tokens, DnD infrastructure, custom Modifier extensions |
| `conversion/` | Format importers: Ableton (`.als`), Apollo (`.approj`) |
| `home/` | Home screen (project browser / launcher) |
| `settings/` | `SettingsDialog` (expect/actual; desktop opens a Swing dialog, Android uses Compose) |
| `desktop/` | Desktop platform features (Discord RPC, FlatLaf LAF, macOS title bar, about handler) |

---

## Signal / Chain Model

Signals (`Signal.LED`) flow through a `Chain` — an ordered list of `GenericChainDevice` instances. Each device:

1. Receives signals via `signalEnter(n: List<Signal>)`
2. Transforms them
3. Forwards via its `signalExit` lambda to the next device

`Chain.reroute()` must be called after the device list changes to rewire the `signalExit` lambdas. `Heaven` sits at the end of the signal path and schedules LED output to physical Launchpad devices.

---

## Device Pattern

Every chain effect is a class that extends `GenericChainDevice<State>` (or `LEDChainDevice<State>` for LED-only effects):

- `State` must be a `@Serializable` data class implementing `DeviceState`
- State is exposed as `MutableStateFlow<State>`
- Mutations call `pushStateChange(before, after)` — this auto-registers an undo action if the state actually changed
- Each device provides a `@Composable fun Content()` for its inline chain editor UI
- The device list and serialization registry live in `DeviceSerializationModule`

---

## Gem System

A **Gem** is a reusable, serializable visual node-graph program:

- `GemAsset` — top-level container (metadata + `GemDefinition`)
- `GemDefinition` — holds one or more `GemGraph` instances; `Gem.rootGraphId = "root"` is the entry point
- `GemGraph` — immutable data class with `nodes: List<GemNodeInstance>` and `connections: List<GemConnection>`; all mutations return a new copy (use `putNode`, `removeNode`, `connect`, `disconnect`, etc.)
- `GemNodeRegistry` — maps type IDs (`GemNodeTypeId`) to `GemNodeDescriptor`s; built-in types are in `GemBuiltInNodes`
- `GemCompiler` / `GemExecutor` — compiles a graph into an execution plan and runs it
- `GemRepository` (in `gem/data/`) — persists assets as JSON
- Schema versioning: `Gem.phase1SchemaVersion = 1.0.0`, `phase4SchemaVersion = 1.1.0`

---

## `expect`/`actual` Convention


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthonyhfm/Amethyst](https://github.com/anthonyhfm/Amethyst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
