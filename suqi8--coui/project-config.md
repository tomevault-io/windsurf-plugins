---
trigger: always_on
description: Compose Multiplatform UI component library. Targets Android, iOS, Desktop (JVM), macOS, Web (Wasm/JS).
---

# Miuix

Compose Multiplatform UI component library. Targets Android, iOS, Desktop (JVM), macOS, Web (Wasm/JS).

> **COUI 改造主线**：本仓库正基于 Miuix 就地演进成一套 COUI（ColorOS 设计语言）组件库。凡涉及「换成 ColorOS 风格 / COUI 换肤 / 调主题或控件视觉 / 逆向提取 ColorOS 设计 token」的任务，先读 `AGENT.md`（设备/工具环境、逆向校准工作流、token→代码落点映射），再按本文件的工程规范落地。本文件的代码风格、API 约定、性能约束在 COUI 改造中**继续生效**。

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
| `coui-core/`           | Utilities + COUIIcons base (depended on by `ui` and `icons`) |
| `coui-ui/`             | Main UI library                                               |
| `coui-preference/`     | Preference / menu / popup components                          |
| `coui-blur/`           | Blur effects (Android minSdk=33)                              |
| `coui-icons/`          | Extended icon resources                                       |
| `coui-navigation3-ui/` | Navigation 3 UI (depends on `coui-squircle` for transition clip) |
| `example/`              | Demo app                                                      |
| `baselineprofile/`      | Android baseline profile generation                           |
| `docs/`                 | VitePress documentation site                                  |
| `build-plugins/`        | Custom Gradle plugins                                         |
| `gradle/`               | Version catalog + wrapper                                     |

### Component Source Layout

`coui-ui/src/commonMain/kotlin/io/github/suqi8/coui/kmp/`:

| Subdir        | Contents                                                        |
| :------------ | :-------------------------------------------------------------- |
| `basic/`      | Fundamental components (Button, Switch, TextField, Surface, …)  |
| `overlay/`    | Scaffold-hosted overlays (OverlayDialog, OverlayBottomSheet, …) |
| `window/`     | Platform window components (WindowDialog, WindowBottomSheet, …) |
| `layout/`     | Shared layout primitives                                        |
| `theme/`      | COUITheme, Colors, TextStyles, ThemeController, DynamicColors  |
| `color/`      | Color utilities, Material Color                                 |
| `anim/`       | Animation utilities                                             |
| `utils/`      | General utilities                                               |
| `icon/`       | Built-in basic icons (ArrowRight, Check, Search, …)             |
| `interfaces/` | Shared interfaces                                               |

`coui-preference/src/commonMain/kotlin/io/github/suqi8/coui/kmp/`:

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


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suqi8/coui](https://github.com/suqi8/coui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
