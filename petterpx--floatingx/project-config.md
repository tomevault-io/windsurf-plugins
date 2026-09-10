---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

FloatingX — an Android floating-window (悬浮窗) library published to Maven Central as
`io.github.petterpx:floatingx-{core,app,scope,system,compose}`. Comments, logs and docs are written
in Chinese; keep that convention when editing library sources.

3.0 is a full rewrite; the 2.x modules (`floatingx/`, `floatingx_compose/`) are gone and the API is
not backwards compatible (see `docs/MIGRATION.md`).

## 模块

所有库模块都用 `build-logic/` 的 `floatingx.library` convention plugin
（minSdk 21 / compileSdk 36 / Java 17 / `explicitApi()` / `jvmDefault=enable` / maven 坐标 / Robolectric）。
模块自己的 `build.gradle.kts` 只声明依赖。

| 模块 | 包 | 内容 | 依赖 |
|---|---|---|---|
| `floatingx-core` | `com.petterp.floatingx.core` | 状态机 `FxEngine`、锚点定位、手势、feature、`FxControl`、`FloatingX` 注册表、`FxSpStorage` | `androidx.annotation`(api)、`androidx.core` |
| `floatingx-app` | `com.petterp.floatingx.app` | `AppHost`：跟随前台 Activity 的全局浮窗；黑白名单 / filter | core、`androidx.core` |
| `floatingx-system` | `com.petterp.floatingx.system` | `SystemHost`：`WindowManager` 窗口、悬浮窗权限、键盘 / 返回键 | core、`androidx.core` |
| `floatingx-scope` | `com.petterp.floatingx.scope` | `ViewGroupHost` / `FragmentHost` 与 `fxScope {}` 局部浮窗 | core、`androidx.fragment`(**compileOnly**) |
| `floatingx-compose` | `com.petterp.floatingx.compose` | `compose {}` DSL、归 control 所有的 `FxComposeOwner`、`stateFlow()` / `positionFlow()`（**minSdk 23**） | core、compose-ui(api)、lifecycle 2.10.0、savedstate、coroutines |
| `app` | `com.petterp.floatingx.demo` | demo + instrumentation 测试工程（minSdk 23） | 以上全部 |

`floatingx-app` 用清单里声明的 `FxAppInitProvider`（ContentProvider）在进程启动时
`FxActivityTracker.init(application)`，所以 install 写在任何时机都能拿到当前前台 Activity。
`floatingx-system` 的清单声明了 `SYSTEM_ALERT_WINDOW` 与权限申请页 `FxPermissionActivity`，
接入方无需自行配置。

依赖边界（CI 有 JUnit 扫描断言）：core 源码不得 import `android.view.WindowManager`、
`androidx.fragment`、`androidx.compose`、`androidx.lifecycle`、`androidx.appcompat`。

## Build & Commands

Java 17 is required (AGP 8.13.2 / Gradle 8.14.3 / Kotlin 2.2.21). Version catalog:
`gradle/libs.versions.toml`.

```bash
./gradlew test                                                       # 全部 JVM/Robolectric 单测（CI 必跑）
./gradlew :floatingx-core:test                                       # 单模块（core 142 / scope 22 / app 32 / system 62 / compose 25 用例）
./gradlew app:assembleDebug                                          # 构建 demo apk
./gradlew app:installDebug                                           # 安装 demo
./gradlew publishToMavenLocal -PisPublish=false -PversionName=3.0.0-SNAPSHOT   # CI 每个 PR 都跑
./gradlew lint                                                       # android lint
```

Instrumentation（需要设备 / 模拟器；CI 用 `reactivecircus/android-emulator-runner` api-level 34 跑
`.github/scripts/instrumentation.sh`——失败自动重跑一次并把 logcat / 截图传成工件；emulator-runner 的
`script` 逐行 `sh -c` 执行，所以逻辑必须放脚本文件里）：

```bash
./gradlew app:installDebug app:installDebugAndroidTest
adb shell appops set com.petterp.floatingx.app SYSTEM_ALERT_WINDOW allow   # 系统浮窗用例需要
adb shell settings put global window_animation_scale 0
adb shell settings put global transition_animation_scale 0
adb shell settings put global animator_duration_scale 0
# 跑完 AGP 会卸载 apk，appops 授权随之丢失；留住 apk 才能免去每次重新授权
./gradlew app:connectedDebugAndroidTest -Pandroid.injected.androidTest.leaveApksInstalledAfterRun=true
```

`ActivityScenario.launch()` 带 `NEW_TASK | CLEAR_TASK`，会把第一页销毁；用例里二级跳页一律用
`TestUtil.kt` 的 `navigateTo()` + `pressBack()`，不用 `ActivityScenario.launch / close`。

**Robolectric 固定 `sdk=35`**（各模块 `src/test/resources/robolectric.properties`）：SDK 36 的沙箱
要求 JDK 21，而本仓库工具链是 JDK 17。改这个值前先确认工具链。

Publishing（release workflow，GitHub Release 触发）：
`./gradlew publishAndReleaseToMavenCentral --no-configuration-cache -PisPublish=true -PversionName=$TAG`

Gradle properties（`settings.gradle` 读进 `rootProject.ext`）：

- `-PversionName` / `-PversionCode` — 默认取 `git describe --tags` 与 `git rev-list HEAD --count`。
- `-PisPublish` — `true` 时应用 `signAllPublications()`（需要 GPG 环境变量）。
- `isDev` in `local.properties`（默认 `true`）— `true` 时 `app` 依赖本地 project，`false` 切到已发布产物。

日志 tag 是 `Fx-<scope>`（`Fx-system` 等）与用户自己传的（demo 用 `Fx-demo`），
所以用 `adb logcat | grep "Fx-"`。只有配置里调过 `enableLog(tag)` 才会有日志。

## Architecture：Host / Engine / Feature

三个正交的角色，取代 2.x 的 Helper → Control → Provider：

1. **Host（`core.host.FxHost`）—— 浮窗挂在哪。**
   `bind(session)` / `createContainer()` / `attach` / `detach` / `bounds()` / `release()`，
   通过 `FxHostSession` 向 engine 报告 `onHostReady` / `onHostLost` / `onBoundsChanged` / `requestSwap`。
   - `AppHost`（app）：容器是 `FxLayerContainer`，挂到当前前台 Activity 的 **DecorView**
     （默认，不是 `R.id.content`，这样拖动才是真正全屏）；换页时把**同一个容器**静默 reparent，
     engine 状态、feature、动画都不重来。被黑白名单/filter 拒绝的页面上整体卸下。
   - `SystemHost`（system）：容器是 `FxWindowContainer`，挂到 `WindowManager`；
     权限三策略 `Auto/Manual/Skip`，被拒时 `requestSwap(fallback)` 降级到 `AppHost`（原 `SYSTEM_AUTO`）。
   - `ViewGroupHost` / `FragmentHost`（scope）：挂到任意 `ViewGroup` / Fragment 根 view；
     不进注册表，生命周期归调用方。

2. **Engine（`core.engine.FxEngine`）—— 状态机 + 命令队列。**
   `INSTALLED → ATTACHED → SHOWN`，终态 `CANCELLED`。host 未 ready 时 `show/hide/moveTo` 入队，

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Petterpx/FloatingX](https://github.com/Petterpx/FloatingX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
