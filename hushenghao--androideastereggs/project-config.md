---
trigger: always_on
description: This document is for AI coding tools. Use it to locate code, choose the right
---

# AndroidEasterEggs Project Map

This document is for AI coding tools. Use it to locate code, choose the right
module, and run the smallest useful verification command.

## Repository Rules

- Keep commit message style consistent with the existing project history.
- Do not revert unrelated working tree changes.
- Prefer focused fixes in the module or method named by the task.
- Use `rg` / `rg --files` for code search.
- For Android changes, verify with the most relevant Gradle module task when possible.

## Maintenance

- Treat source code and Gradle configuration as the source of truth when this
  document conflicts with the project.
- Update this document whenever modules are added, removed, renamed, or their
  responsibilities change.
- Keep this document focused on stable structure, ownership, search targets, and
  verification commands.
- Do not record temporary implementation details, one-off bug context, or
  volatile internal method behavior here.

## Project Shape

This is a multi-module Android project using Kotlin DSL Gradle files.

- Root project name: `Easter Eggs`
- Main application module: `:app`
- Shared Android/Kotlin modules: `:core:*`, `:basic`, `:jvm-basic`
- Feature modules: `:feature:*`
- Android Easter egg implementation modules: `:eggs:*`
- Build convention plugins: `build-logic`
- Utility script modules and scripts: `:script:*`, `script/`

Primary module registration lives in `settings.gradle.kts`.
Dependency versions and plugin aliases live in `gradle/libs.versions.toml`.

### Plugin Inheritance Hierarchy

Convention plugins build on each other in a tiered stack. Each tier
automatically injects certain dependencies:

```
easter.eggs.basic.library     →  :jvm-basic (api)
 ├─ easter.eggs.library       →  :basic (implementation) + Hilt + lintBaseline
 │    └─ easter.eggs.compose.library  →  all of above + Compose BOM/Foundation/UI
 └─ easter.eggs.app           →  all library + Compose + signing + packaging
```

| Plugin ID                      | Module Type | Hilt | Compose | Lint Baseline | Used By |
|-------------------------------|-------------|------|---------|---------------|---------|
| `easter.eggs.basic.library`   | LIBRARY     | No   | No      | No            | `:basic` |
| `easter.eggs.library`         | LIBRARY     | Yes  | No      | Yes           | Older `:eggs:*` (Base–Tiramisu), `:core:custom-tab-browser`, `:core:resources`, `:core:shortcut` |
| `easter.eggs.compose.library` | LIBRARY     | Yes  | Yes     | Yes           | All `:core:*` (except above 3), `:feature:*`, newer `:eggs:*` (UpsideDownCake+) |
| `easter.eggs.app`             | APP         | Yes  | Yes     | No            | `:app` only |

Auto-injected dependencies (no explicit declaration needed in module build file):
- APP and LIBRARY modules → `:basic` (implementation)
- BASIC modules → `:jvm-basic` (api)
- Hilt-enabled → Hilt runtime + compiler (ksp)
- Compose-enabled → Compose BOM + `foundation` + `ui`
- Lint-baseline-enabled → `lint-baseline.xml` in module root

## Version & Environment Constants

| Constant     | Value  | Defined In                                              |
|-------------|--------|---------------------------------------------------------|
| compileSdk  | 37     | `build-logic/convention/src/main/kotlin/Versions.kt`   |
| targetSdk   | 37     | `build-logic/convention/src/main/kotlin/Versions.kt`   |
| minSdk      | 23     | `build-logic/convention/src/main/kotlin/Versions.kt`   |
| buildTools  | 37.0.0 | `build-logic/convention/src/main/kotlin/Versions.kt`   |
| Java        | 17     | `build-logic/convention/src/main/kotlin/Versions.kt`   |
| Kotlin      | 2.4.0  | `gradle/libs.versions.toml`                             |
| AGP         | 9.2.1  | `gradle/libs.versions.toml`                             |
| Hilt        | 2.59.2 | `gradle/libs.versions.toml`                             |
| Compose BOM | 2026.06.00 | `gradle/libs.versions.toml`                         |
| applicationId | `com.dede.android_eggs` | `app/build.gradle.kts`                      |
| versionName | 5.0.1  | `app/build.gradle.kts`                                  |
| versionCode | 77     | `app/build.gradle.kts`                                  |

## Build System

### Build Files

- `build.gradle.kts` — declares top-level plugin aliases, applies `kotlin-gradle-plugin` in buildscript.
- `settings.gradle.kts` — includes all Gradle modules and the `build-logic` included build.
- `gradle/libs.versions.toml` — centralized version catalog for dependencies and plugins.
- `gradle.properties` — JVM args, parallel, configuration-cache, `android.useAndroidX=true`,
  `kotlin.code.style=official`, `android.nonTransitiveRClass=true`.

### Key Gradle Properties

| Property                     | Default | Purpose |
|------------------------------|---------|---------|
| `eggs.androidNext.enable`    | `false` | Conditionally include `:eggs:AndroidNext` module |
| `org.gradle.parallel`        | `true`  | Parallel project execution |
| `org.gradle.configuration-cache` | `true` | Enable configuration cache |
| `android.nonTransitiveRClass` | `true` | Non-transitive R class generation |
| `android.nonFinalResIds`     | `true`  | Non-final resource IDs (legacy eggs compatibility) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hushenghao/AndroidEasterEggs](https://github.com/hushenghao/AndroidEasterEggs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
