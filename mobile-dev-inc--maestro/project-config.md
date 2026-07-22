---
trigger: always_on
description: Shared context for any Claude Code skill or subagent operating in this repo. Skills (`.claude/skills/*`) reference this file rather than restating module roles; if a description here drifts from reality, fix it here once and every skill follows.
---

# AGENTS.md — Maestro

Shared context for any Claude Code skill or subagent operating in this repo. Skills (`.claude/skills/*`) reference this file rather than restating module roles; if a description here drifts from reality, fix it here once and every skill follows.

## Module map

Top-level Gradle modules. Code lives under each module's `src/main/`.

| Module                       | Role                                                                                                                                                                                                                                                                                     |
|------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `maestro-android/`           | On-device Android driver. Kotlin sources compile to two checked-in APKs (`maestro-app.apk`, `maestro-server.apk`) consumed by `maestro-client/`. The build's `copyMaestroAndroid` / `copyMaestroServer` finalizers update those APKs plus a `maestro-android-source.sha256` checksum.    |
| `maestro-ios-driver/`        | Host side of iOS driver wrapper (Kotlin). The actual XCTest runner lives in `maestro-ios-xctest-runner/`.                                                                                                                                                                                |
| `maestro-ios-xctest-runner/` | Swift XCTest runner that runs on the iOS device/simulator. The compiled artifacts (`maestro-driver-ios*.zip`) are checked in under `maestro-ios-driver/src/main/resources/driver-iPhoneSimulator/Debug-iphonesimulator/`.                                                                |
| `maestro-ios/`               | iOS host-side glue (small — most iOS host code lives in `maestro-client/`).                                                                                                                                                                                                              |
| `maestro-client/`            | Host-side Kotlin SDK that drives devices. Platform drivers live in `src/main/java/maestro/drivers/`: `AndroidDriver.kt`, `IOSDriver.kt`, `WebDriver.kt`, `CdpWebDriver.kt`. This is where most "auto-grant", "auto-dismiss", system-dialog handling and platform-specific quirks belong. |
| `maestro-orchestra/`         | Command execution layer. `Orchestra.kt` interprets each Maestro command, applies retries, manages the command lifecycle. Sub-packages: `error/`, `filter/`, `workspace/`, `yaml/`.                                                                                                       |
| `maestro-orchestra-models/`  | Shared command/data models (used by `maestro-orchestra/` and consumers).                                                                                                                                                                                                                 |
| `maestro-cli/`               | CLI entry point + MCP server. Mixed Kotlin (~100 files) + Swift (~56 files for iOS-related CLI bits).                                                                                                                                                                                    |
| `maestro-utils/`             | Shared utilities.                                                                                                                                                                                                                                                                        |
| `maestro-web/`               | Web (browser) driver pieces.                                                                                                                                                                                                                                                             |
| `maestro-proto/`             | Protobuf definitions shared across modules.                                                                                                                                                                                                                                              |
| `maestro-test/`              | Cross-module tests that doesn't require devices.                                                                                                                                                                                                                                         |

## E2E test fixtures (`e2e/`)

Shipped fixtures used by `.github/workflows/test-e2e.yaml`. Run via `e2e/run_tests <android|ios|web>` (see `e2e/run_tests` for env-var inputs `MAESTRO_APP`, `MAESTRO_FLOW_PATH`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mobile-dev-inc/maestro](https://github.com/mobile-dev-inc/maestro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
