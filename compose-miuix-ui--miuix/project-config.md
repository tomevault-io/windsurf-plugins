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

Before committing, run `./gradlew spotlessCheck`; only run `./gradlew spotlessApply` if the check reports violations. CI will reject formatting violations.

## Repository Structure

| Directory               | Purpose                                                       |
| :---------------------- | :------------------------------------------------------------ |
| `miuix-core/`           | Utilities + MiuixIcons base (depended on by `ui` and `icons`) |
| `miuix-ui/`             | Main UI library                                               |
| `miuix-preference/`     | Preference / menu / popup components                          |
| `miuix-blur/`           | Blur effects (Android minSdk=33)                              |
| `miuix-icons/`          | Extended icon resources                                       |
| `miuix-navigation3-ui/` | Navigation 3 UI (depends on `miuix-squircle` for transition clip) |
| `example/`              | Demo app                                                      |
| `baselineprofile/`      | Android baseline profile generation                           |
| `docs/`                 | VitePress documentation site                                  |
| `build-plugins/`        | Custom Gradle plugins                                         |
| `gradle/`               | Version catalog + wrapper                                     |

### Component Source Layout

`miuix-ui/src/commonMain/kotlin/top/yukonga/miuix/kmp/`:

| Subdir        | Contents                                                        |
| :------------ | :-------------------------------------------------------------- |
| `basic/`      | Fundamental components (Button, Switch, TextField, Surface, …)  |
| `overlay/`    | Scaffold-hosted overlays (OverlayDialog, OverlayBottomSheet, …) |
| `window/`     | Platform window components (WindowDialog, WindowBottomSheet, …) |
| `layout/`     | Shared layout primitives                                        |
| `theme/`      | MiuixTheme, Colors, TextStyles, ThemeController, DynamicColors  |
| `color/`      | Color utilities, Material Color                                 |
| `anim/`       | Animation utilities                                             |
| `utils/`      | General utilities                                               |
| `icon/`       | Built-in basic icons (ArrowRight, Check, Search, …)             |
| `interfaces/` | Shared interfaces                                               |

`miuix-preference/src/commonMain/kotlin/top/yukonga/miuix/kmp/`:

| Subdir        | Contents                                                        |
| :------------ | :-------------------------------------------------------------- |
| `preference/` | Preference components (SwitchPreference, CheckboxPreference, …) |
| `menu/`       | Dropdown menus (Overlay/Window variants, cascading)             |
| `popup/`      | Dropdown popup primitives shared by menus                       |

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

- **Formatter**: Spotless + ktlint with Compose rules (`io.nlopez.compose.rules:ktlint`). Exact versions live in `build-plugins/src/main/kotlin/module.spotless.gradle.kts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compose-miuix-ui/miuix](https://github.com/compose-miuix-ui/miuix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
