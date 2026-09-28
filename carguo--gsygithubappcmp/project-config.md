---
trigger: always_on
description: > **本项目是 Compose Multiplatform，5 端共享 commonMain UI，agent 改任何 feature 默认改 commonMain。**
---

# 项目定位（Project Identity - 最高优先级，先读这一段再动手）

> **本项目是 Compose Multiplatform，5 端共享 commonMain UI，agent 改任何 feature 默认改 commonMain。**

- rootProject 名：`GSYGithubAppCompose`
- 5 个 KMP target：`androidTarget()` / `jvm("desktop")` / `iosArm64()` / `iosSimulatorArm64()` / `macosArm64()`
- 3 个入口模块：`androidApp/`、`desktopApp/`、`iosApp/`，bundle id 统一为 `com.shuyu.gsygithubappcompose`
- 13 个 feature 模块（[feature/code](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/code)、[detail](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/detail)、[dynamic](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/dynamic)、[history](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/history)、[home](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/home)、[info](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/info)、[issue](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/issue)、[list](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/list)、[login](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/login)、[notification](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/notification)、[profile](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/profile)、[push](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/push)、[search](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/search)、[trending](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/trending)、[welcome](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/welcome)）的 UI/ViewModel **全部位于 `commonMain`**。`androidMain` / `jvmMain` / `iosMain` / `macosMain` 只放平台胶水（`expect/actual`、Toast、SystemUi、文件路径、Native interop 等），不再承载业务 UI。
- 共享底座：[core/common](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common)、[core/ui](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/ui)、[core/network](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/network)、[core/database](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/database)、[data](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/data)、[composeShared](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/composeShared)（iOS/macOS 接入点 `MainViewController.kt`）。

## Ground Truth 版本表（与 [gradle/libs.versions.toml](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/gradle/libs.versions.toml) 必须对齐）

| 组件 | 版本 |
| --- | --- |
| Compose Multiplatform | **1.10.3** |
| Kotlin | **2.3.20** |
| AGP | **9.0.0** |
| Ktor | **3.1.0** |
| Koin | **4.2.1** |
| Coil | **2.7.0** |
| jetbrains-navigation-compose | **2.9.1** |

> 改 build.gradle.kts、写新代码时一律以这张表为准。**禁止**临时升级/降级到其他版本"试试看"。

---

# 1. 默认改动落点：commonMain（不要默认 androidMain-only）

**铁律：任何 feature 的功能改动、Bug 修复、UI 调整，默认改 [commonMain](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature)，不要只改 androidMain。**

- ✅ 在 `feature/<name>/src/commonMain/kotlin/...` 改 `XxxScreen.kt` / `XxxViewModel.kt`
- ❌ 看到 androidMain 有同名文件就直接改 androidMain（90% 情况是误判，那只是一个胶水）
- ❌ 把 commonMain 已有的逻辑复制到 androidMain 再改

**只允许改 androidMain（platform-specific）的场景**：
1. Android 专属 OAuth Redirect / Deep Link / Intent / Push 通道
2. Android `actual` 实现（如 `Dispatchers.android.kt`、平台 SystemUi、Toast、Clipboard）
3. `androidMain/res/`、`AndroidManifest.xml`、`androidApp/` 入口、`proguard-rules.pro`
4. KSP 仍未支持 KMP 的模块（如 [core/database](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/database) 中的 Room）

> 若一个修改在某 target 上不可避免地依赖该平台 API（例如 `androidx.compose.foundation.*` 的 `LocalOverscrollFactory`、`android.content.Context`、`androidx.activity.*`），**必须在 PR/commit 描述里明确标注 "platform-specific (androidMain/jvmMain/iosMain/macosMain)"，并解释为什么不能下沉到 commonMain**。

## 常见 platform-specific API 速查（出现这些 import 立刻警觉）

| import / API | 仅可用于 |
| --- | --- |
| `androidx.compose.foundation.*`（Android 限定的，例如 `LocalOverscrollFactory`、`PullRefreshIndicator` 老 API） | androidMain |
| `androidx.compose.material.*`（旧 Material 1，CMP 已统一 Material 3） | 通常仅 androidMain；commonMain 用 `androidx.compose.material3.*` |
| `androidx.activity.*` / `ComponentActivity` / `BackHandler`（旧版） | androidMain；commonMain 用 `androidx.compose.ui.backhandler.BackHandler`（CMP 1.7+） |
| `android.content.Context` / `@StringRes` / `android.util.Log` / `Toast.makeText` | androidMain |
| `java.io.File` / `java.text.SimpleDateFormat` / `java.util.*` | jvmMain / androidMain；commonMain 用 `kotlinx.datetime`、`okio.Path` |
| `platform.UIKit.*` / `platform.Foundation.*` | iosMain / macosMain |
| `androidx.datastore` 直接用 Context 创建 | androidMain；commonMain 用 KMP DataStore + `expect fun preferenceDataStore(...)` |

> 若 commonMain 必须用到上述能力，标准做法是：在 commonMain 写 `expect`，在 4 个平台 sourceSet 各写一个 `actual`。**不允许**用 `Platform.isAndroid` 这种运行时分支绕过 expect/actual。

---

# 2. Source Set 矩阵：每个 source set 能用什么

> 严格按这张表写代码。任何越界（如 commonMain 出现 Android API）必须立刻通过 expect/actual 修正。

| Source Set | 可用 API | 不可用 API | 典型职责 |
| --- | --- | --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CarGuo/GSYGithubAppCMP](https://github.com/CarGuo/GSYGithubAppCMP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
