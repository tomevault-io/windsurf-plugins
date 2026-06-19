---
trigger: always_on
description: Build and migrate Android apps with Kotlin, Jetpack Compose, MVVM, Hilt, Room 3 (KSP, SQLiteDriver, Flow/suspend DAOs), Navigation3, and multi-module Gradle. Use for new projects or modules, Compose screens and ViewModels, Room 3 and RemoteMediator, API 37 / targetSdk migration, Play Integrity client wiring, offline-first sync, and version-catalog alignment. Not for iOS, Flutter, React Native, KMP-only shared code without an Android app module, or backend-only APIs with no Android client.
---

# Android Kotlin Compose Development

**Context ladder (smaller load first; full references stay complete):**

1. This file (`SKILL.md`) - routing, stop rules, examples.
2. `references/*-quick.md` when listed below - required/forbidden + section links (~40 lines).
3. One target section in the full `references/*.md` - code samples and checklists only.
4. [INDEX-sections.md](references/INDEX-sections.md) - anchor dump only when quick routing is insufficient.

Forbidden: load [INDEX-sections.md](references/INDEX-sections.md) or an entire multi-thousand-line reference when one section or a quick file covers the task.

Route tasks through the Quick Reference table. When no row matches, or the task needs greenfield bootstrap: [workflows.md](references/workflows.md). Full file list: [INDEX.md](references/INDEX.md).

**Required:**

- **Existing project:** read `settings.gradle.kts`, `gradle/libs.versions.toml`, and the `app` module build file before copying from `assets/` - [dependencies.md](references/dependencies.md#existing-project-brownfield), [modularization.md](references/modularization.md#existing-project-alignment). Stack migrations: [migration.md](references/migration.md).
- **Greenfield:** [workflows.md](references/workflows.md) → "Creating a new project?"
- After module, DI, navigation, Room schema, or AGP/Kotlin/KSP changes: `./gradlew help` then `:app:assembleDebug` (use the real app module name) - [gradle-setup.md](references/gradle-setup.md#verify-after-toolchain-or-module-changes).

**Outside-repo stop rules (do not substitute repo edits):** Play upload, tracks, rollout, `versionCode` - [android-ci-cd.md](references/android-ci-cd.md); Play Integrity prerequisites (Console/Cloud setup) - [android-security-quick.md](references/android-security-quick.md); production `adb install` / `pm clear` - [testing.md](references/testing.md#agent-automation-adb-and-uiautomator).

## Quick Reference

Rare or niche topics not listed here are in [INDEX.md](references/INDEX.md) (complete file list).

| Task                                                                                                           | Reference                                                                                                                                                                                                     |
|----------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Task not in table, greenfield bootstrap, multi-topic setup                                                     | [workflows.md](references/workflows.md)                                                                                                                                                                       |
| Full index of all reference files                                                                              | [INDEX.md](references/INDEX.md)                                                                                                                                                                               |
| Version catalog, pins, alpha policy, brownfield alignment                                                      | [dependencies.md](references/dependencies.md#version-strategy)                                                                                                                                                |
| Adding or updating dependencies (catalog aliases)                                                              | [dependencies.md](references/dependencies.md)                                                                                                                                                                 |
| Multi-module dependencies                                                                                      | [dependencies.md](references/dependencies.md)                                                                                                                                                                 |
| Project structure and modules                                                                                  | [modularization.md](references/modularization.md)                                                                                                                                                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Drjacky/claude-android-ninja](https://github.com/Drjacky/claude-android-ninja) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
