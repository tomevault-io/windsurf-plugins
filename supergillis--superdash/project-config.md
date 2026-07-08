---
trigger: always_on
description: A predictable place for the conventions this codebase relies on that the tooling
---

# Contributor & Agent Guide

A predictable place for the conventions this codebase relies on that the tooling
cannot enforce. These apply to human contributors and AI coding agents alike.

- For install and run instructions, see [README.md](README.md).
- For design and module detail, see [docs/architecture/README.md](docs/architecture/README.md).
- For contribution basics, see [CONTRIBUTING.md](CONTRIBUTING.md).

## Project Overview

superdash is a Home Assistant kiosk app for Android tablets. It runs a Home
Assistant dashboard full-screen, stays signed in across restarts, and adds
wall-panel extras: on-device wake word and voice, an ambient photo screensaver,
and doorbell camera overlays. It can expose itself back to Home Assistant over
the ESPHome native API.

Multi-module Gradle/Kotlin build. The app module is `:packages:app`.

| Package | Purpose |
|---|---|
| `packages/app` | Android app, UI, wiring, services, settings. |
| `packages/core` | Shared logging and small utilities. |
| `packages/ha-client` | Home Assistant OAuth, tokens, WebSocket, Assist, media source. |
| `packages/voice` | Wake word, on-device STT (Whisper/Moonshine), local intents. |
| `packages/screensaver` | Screensaver and Immich photo slideshow. |
| `packages/doorbell` | Doorbell camera overlay. |
| `packages/esphome-server` | ESPHome native API server and mDNS announce. |
| `packages/immich-client` | Immich API client for slideshow photos. |
| `packages/kiosk-bus` | Internal event bus. |

Per-package orientation lives in [packages/README.md](packages/README.md).

## Environment

- JDK 17 on `PATH`.
- Android SDK installed. The build resolves NDK and CMake components as needed.
- Native code is part of the build. No prebuilt `.so` is checked in.
- `whisper.cpp` is fetched on demand and is optional. CMake builds a stub when
  it is absent, so a plain checkout builds fine without it.
- Git LFS is not required.

## Build, Test, Lint

```bash
./gradlew :packages:app:assembleDebug          # build the app
./gradlew testDebugUnitTest                     # run all unit tests
./gradlew :packages:app:testDebugUnitTest       # app module tests
./gradlew :packages:ha-client:testDebugUnitTest # ha-client module tests
./gradlew :packages:voice:testDebugUnitTest     # voice module tests
./gradlew :packages:esphome-server:testDebugUnitTest
./gradlew :packages:immich-client:testDebugUnitTest
./gradlew :packages:core:testDebugUnitTest
./gradlew ktlintCheck                            # lint check (gates)
./gradlew ktlintFormat                           # auto-format
```

## How to Verify a Change

A change is done when:

- `./gradlew ktlintCheck` passes.
- The relevant module unit tests pass.
- New behavior has a test, and it is a unit test unless it needs a device.

Prefer focused module tests over a full `./gradlew assembleDebug` during
refactors. Run `./gradlew ktlintCheck testDebugUnitTest` before opening a PR.

## Documentation Style

- Keep docs short.
- Prefer bullets over long paragraphs.
- Use small tables for maps and lists.
- Do not use em dash.
- Do not keep historical notes in active docs.
- Link only to existing files.
- Use current paths under `packages/`.
- Use `superdash` for the app name.

## Code Style

### Braces

Always brace `if`, `else`, and `when` bodies.

Good:

```kotlin
if (bytes.isEmpty()) {
    return null
}
```

Bad:

```kotlin
if (bytes.isEmpty()) return null
if (bytes.isEmpty()) { return null }
```

For expression values, split to multiline form.

```kotlin
val colors =
    if (darkTheme) {
        darkColorScheme()
    } else {
        lightColorScheme()
    }
```

### Names

Use descriptive names.

Avoid:

- `v`
- `q`
- `s`
- `k`

Allowed:

- Math coordinates like `x` and `y`.
- Loop indexes like `i` and `j`.
- Type parameters like `T`, `R`, `K`, `V`.
- `it` when the lambda is clear.
- `e` or `t` for caught throwables.

### Tests

Use backtick test names with spaces.

Good:

```kotlin
@Test fun `long pause after wake times out`() {}
```

Bad:

```kotlin
@Test fun long_pause_after_wake_times_out() {}
```

Android instrumentation test method names must be dex-safe camelCase. Do not use
backtick names with spaces in `src/androidTest`.

### Comments

Default to no comment.

Write a comment only for:

- Hidden constraints.
- Workarounds.
- Non-obvious why.
- Behavior that may surprise a reader.

Do not restate the code.

### JSON

Prefer typed `@Serializable` data classes.

Use `JsonElement` only for dynamic shapes.

Examples:

- HA entity attributes.
- Pass-through payloads.
- Unknown framed payloads.

### ktlint

- `./gradlew ktlintCheck` gates checks.
- `./gradlew ktlintFormat` can rewrite files.
- Do not add new rule disables without a one-line reason.
- Production line length is 120.
- Test source sets may use longer fixtures.

## Logging

Use one `Log` per file that logs.

```kotlin
private val log = Log("MyClass")
```

Use structured fields when there is more than one data point.

```kotlin
log.i("seeded entities", "count" to count)
log.w("connection failed", null, "reason" to reason)
log.w("auth invalid", throwable)
```

MUST use structured fields instead of interpolating key/value details into the message.

Good:

```kotlin
log.i("onReceive", "action" to action)
```

Bad:

```kotlin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [supergillis/superdash](https://github.com/supergillis/superdash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
