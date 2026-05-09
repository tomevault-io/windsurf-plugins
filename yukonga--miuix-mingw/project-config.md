---
trigger: always_on
description: Compose Multiplatform UI component library. Targets Android, iOS, Desktop (JVM), macOS, Web (Wasm/JS).
---

# Miuix

Compose Multiplatform UI component library. Targets Android, iOS, Desktop (JVM), macOS, Web (Wasm/JS).

## Quick Start

- For significant features or refactors, sketch an Plan first; keep it updated as you work.
- Run the component-specific checks below before handing work off; do not skip failing steps.
- Use Context7 to pull library/API docs when you touch unfamiliar Compose/Android/JVM/Js/WasmJs/Swift APIs or deps.
- Always use Chinese to communicate with users and output Plan content, but the generated code (including comments and KDoc) must remain in English.

## Key Commands

| Action              | Command                                                 |
| :------------------ | :------------------------------------------------------ |
| Build (full)        | `./gradlew assemble`                                    |
| Build (quick check) | `./gradlew compileKotlinDesktop`                        |
| Test                | `./gradlew check`                                       |
| Check formatting    | `./gradlew spotlessCheck`                               |
| **Fix formatting**  | `./gradlew spotlessApply`                               |
| Run Android demo    | `./gradlew :example:android:installDebug`               |
| Run Desktop demo    | `./gradlew :example:desktop:hotRunDesktop --auto`       |
| Run WasmJs demo     | `./gradlew :example:web:wasmJsBrowserRun`               |
| Run Js demo         | `./gradlew :example:web:jsBrowserDevelopmentRun`        |
| Run macOS demo      | `./gradlew :example:macos:runDebugExecutableMacosArm64` |
| Run iOS demo        | Open `example/ios/iosApp.xcodeproj` in Xcode and run    |

**ALWAYS run `./gradlew spotlessApply` before committing.** CI will reject formatting violations.

## Repository Structure

| Directory                | Purpose                                                        |
| :----------------------- | :------------------------------------------------------------- |
| `miuix-core/`           | Core utilities and MiuixIcons base; depended on by `miuix-ui` and `miuix-icons` |
| `miuix-ui/`             | Main UI library — basic components, theme, colors, animations, overlay/window/layout |
| `miuix-preference/`     | Preference components (SwitchPreference, CheckboxPreference, etc.)                  |
| `miuix-blur/`           | Blur/backdrop effects (Android minSdk=31)                                           |
| `miuix-icons/`          | Extended icon resources                                                             |
| `miuix-shapes/`         | Smooth rounded corner shapes (SmoothRoundedCornerShape, SmoothCapsuleShape, etc.)   |
| `miuix-navigation3-ui/` | Navigation 3 UI implementation (independent, no miuix dependency)                   |
| `example/`              | Demo app — showcases and tests all components                                       |
| `baselineprofile/`      | Android baseline profile generation                                                 |
| `docs/`                 | VitePress documentation site                                                        |
| `build-plugins/`        | Custom Gradle plugins for build logic reuse                                         |
| `gradle/`               | Version catalog (`libs.versions.toml`) and wrapper                                  |

### Component Source Layout

```
miuix-ui/src/commonMain/kotlin/top/yukonga/miuix/kmp/
├── basic/       # Fundamental components (Button, Switch, TextField, Surface, etc.)
├── overlay/     # Scaffold-hosted overlay components (OverlayDialog, OverlayBottomSheet, etc.)
├── window/      # Platform window components (WindowDialog, WindowBottomSheet, etc.)
├── layout/      # Shared layout components (DialogContentLayout, BottomSheetContentLayout, etc.)
├── theme/       # MiuixTheme, Colors, TextStyles, ThemeController, SmoothRounding, DynamicColors, etc.
├── color/       # Color utilities, Material Color
├── anim/        # Animation utilities
├── utils/       # General utilities
├── icon/        # Built-in basic icons (ArrowRight, Check, Search, etc.)
└── interfaces/

miuix-preference/src/commonMain/kotlin/top/yukonga/miuix/kmp/
└── preference/  # Preference components (SwitchPreference, CheckboxPreference, ArrowPreference, etc.)
```

### Platform Source Sets

```
commonMain
├── androidMain
└── skikoMain (Skiko rendering — all non-Android targets)
    ├── darwinMain (iOS + macOS)
    │   ├── iosMain
    │   └── macosMain
    ├── desktopMain (JVM)
    └── webMain
        ├── wasmJsMain
        └── jsMain
```

99% of UI logic lives in `commonMain`. Only use platform source sets for genuinely platform-specific code.

## Code Style

- **Formatter**: Spotless + ktlint 1.8.0 with Compose rules (`io.nlopez.compose.rules:ktlint:0.5.6`)
- **License header** (required on all `.kt` and `.kts` files):

  ```
  // Copyright $YEAR, compose-miuix-ui contributors
  // SPDX-License-Identifier: Apache-2.0
  ```

  Spotless auto-fills `$YEAR` with the current year. Do not manually change years in existing file headers.
- **Spotless exclusions**: Icon files (`**/icon/**/*.kt`) and navigation3 files (`**/navigation3/**/*.kt`) are excluded from formatting.
- Line endings: platform-native

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YuKongA/miuix-mingw](https://github.com/YuKongA/miuix-mingw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
