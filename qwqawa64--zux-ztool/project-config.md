---
trigger: always_on
description: - Default to `rg` for searching and keep edits ASCII unless the file already uses non-ASCII, especially when touching old .java files, they may contain Chinese logging strings and hardcoded UI strings.
---

# AGENTS.MD

## Agent Quick Start

- Default to `rg` for searching and keep edits ASCII unless the file already uses non-ASCII, especially when touching old .java files, they may contain Chinese logging strings and hardcoded UI strings.

## Project Overview

ZUX-ZTool is an Android/Xposed tool project. The current refactor direction is a UI-layer migration from traditional XML/View/Fragment screens to Jetpack Compose, while preserving Hook, service, utility, configuration, and runtime asset behavior.

The preferred boundary is:

- Refactor UI, navigation, state handling, dialogs, and theme/design-system code.
- Keep `hook/**`, Xposed metadata, module assets, shell behavior, service behavior, existing preference keys, and existing launch entry points intact unless explicitly requested.
- Do not turn the migration into a mixed View/Compose layout swap when a screen contains business logic that should first be separated into state and reusable logic.

## Repository Shape

Root:

```text
ZUX-ZTool/
  app/                         Android application module
  gradle/                      Gradle wrapper files
  libs/                        root-level local dependencies
  build.gradle.kts             root build script
  settings.gradle.kts          Gradle module configuration
  gradle.properties            Gradle/Android build settings
  ComposeRefactor.md           historical Compose refactor notes, UTF-8 encoding
  README.md
  UpdateCheck.json
  ZToolLogo.png
  ZToolLogoForeground.svg
  更新日志.txt
```

Main application areas:

```text
app/src/main/java/com/qimian233/ztool/
  MainActivity.kt
  HomeFragment.kt / HomeFragment.java history
  FeaturesFragment.kt
  SettingsFragment.kt
  AuditFragment.kt / AuditFragment.java history
  EnhancedShellExecutor.java
  LoadingDialog.kt

  audit/
  config/
  hook/
  service/
  settingactivity/
  ui/
    components/
    theme/
  utils/
```

Important resources and assets:

```text
app/src/main/res/
  drawable/
  mipmap-anydpi-v26/
  values/
  values-night/
  raw/
  xml/

app/src/main/assets/
  xposed_init
  embedding/
```

## Architecture Direction

Use Compose as the long-term UI layer.

- Use `ComposeRefactor.md` as the detailed migration plan and status log. Before selecting the next migration target, consult the latest plan and verification notes there; keep this file as the concise operating guide.
- `MainActivity` should move toward only hosting `setContent { ZToolApp() }`.
- Screens should become composable screen implementations.
- XML Navigation has been replaced by `navigation-compose`; do not reintroduce XML navigation.
- Existing `settingactivity/**` Activities may be migrated gradually, but new UI should not expand the old one-Activity-per-setting pattern unless compatibility requires preserving an entry point.
- Business pages should consume stable state rather than directly running shell commands, starting raw threads, showing imperative dialogs, or reading/writing preferences inline.
- Phase 7 cleanup may replace temporary Fragment/AppCompat/dialog compatibility wrappers after their Hook or API compatibility target has a stable replacement.

Preferred state shape:

- `ViewModel`
- `UiState`
- `StateFlow`
- Repository/Manager wrappers for shell, logs, config, preferences, update checks, and system services.

For example, logic from `HomeFragment` should be modeled around:

- `HomeViewModel`
- `HomeUiState`
- `EnvironmentRepository`
- `UpdateRepository`

For Phase 7, avoid adding new dependencies on `HomeFragment`; use the replacement Hook compatibility target documented in `ComposeRefactor.md` once it exists.

## Compose And Design-System Rules

Phase 3 is centered on a project-level theme settings model and a component adapter layer. Business screens should stay style-agnostic and consume project components.

Theme settings should be modeled and persisted outside feature screens. The durable shape should cover:

- Frontend style: Material 3 Expressive, Miuix, and future styles.
- Theme mode: follow system, light, and dark.
- Android 12+ Monet dynamic color.
- Manual seed color.
- AMOLED pure black mode.

Keep theme preference keys scoped to app UI preferences. Do not reuse or alter Hook/module preference keys for theme UI settings.

`ZToolTheme` should resolve the final theme from settings:

- Use Monet only when dynamic color is enabled, the device supports it, and manual color is disabled.
- Manual color should derive a complete Material 3 `ColorScheme`, not only replace `primary`.
- AMOLED black should be a dark-theme post-processing step that consistently overrides `background`, `surface`, and `surfaceContainer*`.
- Semantic data colors, such as log severity and user-selected color previews, may remain explicit.

Avoid placing frontend-style conditionals directly in business screens.

Do not scatter code like:

```kotlin
if (style == FrontendStyle.Miuix) { ... }
```

inside feature screens. Instead, prefer project-level UI components and theme adapters:

- `ZToolTheme`
- `ZToolScaffold`
- `ZToolTopAppBar`
- `ZToolNavigationRail`
- `ZToolCard`
- `ZToolDropdownField`
- `ZToolSwitchRow`
- `ZListItem`
- `ZDialog`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qwqawa64/ZUX-ZTool](https://github.com/qwqawa64/ZUX-ZTool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
