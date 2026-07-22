---
trigger: always_on
description: ComposeHooks 是一个 Kotlin Multiplatform 库，在 Jetpack Compose 中提供 React 风格的 Hooks。灵感来自 [alibaba/hooks](https://github.com/alibaba/hooks)。
---

# Repository Guidelines

## Project Overview

ComposeHooks 是一个 Kotlin Multiplatform 库，在 Jetpack Compose 中提供 React 风格的 Hooks。灵感来自 [alibaba/hooks](https://github.com/alibaba/hooks)。

**支持平台**: Android, Desktop (JVM), iOS (arm64, x64, simulator-arm64), Web (wasmJs browser，自 2.4.0-beta-1)

**Artifact**: `xyz.junerver.compose:hooks2:<version>`

## Project Structure & Module Organization

ComposeHooks is split into the `hooks` and `ai` Kotlin Multiplatform libraries plus the `app` multiplatform showcase client. Library sources live in `hooks/src/*` with platform folders like `commonMain`, `commonTest`, `androidMain`, `desktopMain`, the `ios*` targets, and `wasmJsTest`. The `ai` module ships AI-oriented hooks (useAgent / multiprovider / tokenstats) and mirrors the same KMP targets. Example screens and demo scaffolding reside under `app/src/...`, mirroring the platform folders, while shared assets and docs sit in `art/` and `docs/`. Gradle wiring remains at the root (`build.gradle.kts`, `settings.gradle.kts`, `gradle.properties`) alongside contributor resources.

```
hooks/src/
├── commonMain/kotlin/xyz/junerver/compose/hooks/
│   ├── userequest/          # 网络请求管理 (插件架构)
│   ├── useform/             # 表单验证框架
│   ├── useref/              # Ref 相关 hooks
│   └── *.kt                 # 各种 hooks (useState, useEffect, useReducer 等)
├── commonJvmAndroid/        # JVM+Android 共享代码 (kotlin-reflect 实际调用隔离于此)
├── androidMain/             # Android 专属 hooks (useBiometric, useNetwork 等)
├── desktopMain/             # Desktop 专属代码
├── iosMain/                 # iOS 专属代码
├── commonTest/              # 跨平台测试 (kotlin.test + kotlinx-coroutines-test)
├── desktopTest/             # Desktop 专属测试 (含 Compose UI 测试)
└── wasmJsTest/              # wasmJs target 冒烟测试 (继承 commonTest，验证 WasmJsSmokeTest)

ai/src/                      # AI hooks 库 (useAgent / multiprovider / tokenstats)
├── commonMain, androidMain, desktopMain, iosMain, wasmJsMain

app/src/
├── commonMain/              # 示例代码，展示各 hook 用法
├── wasmJsMain/              # Web 示例入口 (main.kt + ComposeViewport + CJK 字体)
└── (androidMain, desktopMain, iosMain)
```

## Build, Test, and Development Commands

> **Windows 平台注意：** 使用 `.\gradlew.bat` 代替 `./gradlew`，例如 `.\gradlew.bat build`、`.\gradlew.bat :hooks:jvmTest` 等。下表以 Linux/macOS 风格为例。

```bash
# 构建
./gradlew build
./gradlew :hooks:build       # 仅构建库

# 格式化代码 (提交前必须运行)
./gradlew formatKotlin

# 检查格式
./gradlew lintKotlin

# 质量门禁 (CI 使用，见 .github/workflows/ci.yml)
./gradlew :hooks:runQualityChecks       # 格式 + 静态检查
./gradlew :hooks:runCoverageChecks      # 测试覆盖率门禁
./gradlew :hooks:verifyReleaseReadiness # 上述两项的合集

# 测试
./gradlew test                          # 单元测试
./gradlew desktopTest                   # Desktop 测试
./gradlew :hooks:wasmJsBrowserTest      # wasmJs 浏览器测试 (需 Chrome)
./gradlew :hooks:check                  # 完整检查
./gradlew androidInstrumentedTest       # Android 插桩测试

# 运行示例应用
./gradlew :app:run                      # Desktop
./gradlew :app:installDebug             # Android
./gradlew :app:wasmJsBrowserDevelopmentRun  # Web (wasmJs) 本地开发服务器

# 发布
./gradlew :hooks:publishToMavenLocal    # 本地验证
./gradlew publishToMavenCentral         # 正式发布
```

## Coding Style & Naming Conventions

Indent with four spaces and keep Kotlin lines under 140 characters as enforced by `.editorconfig`. Use Kotlin Official formatting with trailing commas enabled and never introduce wildcard imports. Include the standard header comment block in every new Kotlin file and favor descriptive, imperative function names.

## Hook Development Standards

### 命名和返回值
- Hook 函数名以 `use` 开头，如 `useNetwork`
- 返回值类型命名为 `XxxHolder`
- 所有 `use` 函数都有对应的 `remember` 签名别名

### 实现规范
- 不直接返回状态值，包装在 `State` 中
- Holder 中 `State` 放在前面，函数放在后面
- 优先使用现有 hooks 而非原生 Compose 函数：
  - `useState` 代替 `derivedStateOf`
  - `useCreation` 或 `useRef` 代替 `remember`
  - `useEffect` 代替 `LaunchedEffect`
- 函数成员声明类型别名

### useRequest 插件系统
位于 `userequest/` 目录，核心功能通过插件实现：缓存、防抖、节流、重试、轮询等。

## Testing Guidelines

Add shared coverage in `hooks/src/commonTest/kotlin` with `kotlin.test` and `kotlinx.coroutines.test`; desktop-only verifications belong in `hooks/src/desktopTest`; wasmJs target coverage lives in `hooks/src/wasmJsTest` (it inherits commonTest and adds a smoke test for createStore/reducer/combineStores/middleware). UI behavior should use Compose testing APIs in the relevant platform source set. Run `./gradlew :hooks:check` before opening a PR, and complement feature work with focused tests. Android instrumentation lives in `hooks/src/androidInstrumentedTest` and can be executed on a device or emulator via `./gradlew :hooks:connectedDebugAndroidTest`. The CI workflow (`.github/workflows/ci.yml`) gates merges on `:hooks:runQualityChecks`, `:hooks:runCoverageChecks`, and `:hooks:build`.

### TDD Practice Requirements

Tests exist to discover implementation defects, not merely to pass. Follow these principles:

1. **Write tests that challenge the implementation**: Design test cases based on expected behavior and edge cases, not by mimicking the implementation logic. Avoid "copy-paste" tests that simply mirror what the code does.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junerver/ComposeHooks](https://github.com/junerver/ComposeHooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
