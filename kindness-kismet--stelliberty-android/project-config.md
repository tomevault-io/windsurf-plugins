---
trigger: always_on
description: miuix + mihomo 的 Android 代理客户端。单模块 `:app`（`com.android.application`，AGP 9 内置 Kotlin，源码全在 `src/main`）+ `:baselineprofile`（`com.android.test`，只产 Baseline Profile 不进 APK）。UI 用 AndroidX Compose，miuix 走其 `-android` 发布件。
---

# Stelliberty

miuix + mihomo 的 Android 代理客户端。单模块 `:app`（`com.android.application`，AGP 9 内置 Kotlin，源码全在 `src/main`）+ `:baselineprofile`（`com.android.test`，只产 Baseline Profile 不进 APK）。UI 用 AndroidX Compose，miuix 走其 `-android` 发布件。

本文件是 agent 指南的入口，只讲**项目结构**与**跨子系统都成立的规范**——每次会话都会全量加载，
所以放不下的东西不该放这里。子系统各自的约束在 skill 里按需读，见末尾分派表。

## 工作规程

- 每次改动至少跑 `git diff --check` + 与变更匹配的编译验证。
- 首次编译或资源缺失时运行 `python scripts/prebuild.py`，恢复未跟踪的 Gradle 启动器、便携 JDK / Go 与 GeoIP。
- 编译一律走 `python scripts/build.py`。脚本默认编译 release，`--dev` 编译 debug；助手未指定构建类型时始终加 `--dev`。
- 改 Kotlin：`python scripts/build.py compile --dev`。验证 native / 打包：`python scripts/build.py --dev`。架构 `--abi`，默认 `arm64-v8a`。
- 单元测试在 `android/app/src/test/`，改到被测代码时跑 `python scripts/build.py gradle :app:testDebugUnitTest`。
- 新增 composable 后临时加 `composeCompiler { reportsDestination.set(layout.buildDirectory.dir("compose_reports")) }`，再 `python scripts/build.py gradle :app:compileDebugKotlin --rerun-tasks` 跑报告，确认 restartable 全部 skippable、0 unstable 参数（当前 132 个），验完删掉临时配置。
- `third_party/mihomo` 是 submodule、`third_party/scripta` 是 includeBuild 复合构建，改前先确认确需触及。
- 保留用户已有的未提交改动；不用破坏性 reset/checkout；不修改或输出 `local.properties`。
- 完成后先报告变更与验证结果。**未经用户明确授权，不执行 `git add`/`commit`，不创建或修改远程 PR**；要求撰写文案不代表授权执行这些操作。
- **禁止擅自 `git push`**：先说明全部待推送提交、验证结果、远端和目标分支，再取得明确确认。提交或创建 PR 的授权不包含推送，也不能借 `gh pr create` 隐式推送；会话中已确认且范围、目标未变的推送无需重复询问。
- **Commit 主题与正文、PR 标题与正文一律使用英文**。撰写或执行提交先阅读 `commit`，准备或创建、更新 PR 先阅读 `pr`；格式、范围与验证规则由对应技能维护。
- Git 使用目录与后缀白名单。新增编译输入须核对 `.gitignore`；可下载产物不跟踪，Baseline Profile 必须保留。

## 技术栈

Kotlin（AGP 9 内置，不加独立 kotlin 插件）。UI：Compose（经 miuix `-android` 件传递）+ miuix（含自带 NavDisplay）+ androidx navigationevent（预测性返回手势）。图标是 `ui/icon/` 下手写的 MingCute ImageVector，**没有** material-icons 依赖。数据：JSON 文件（kotlinx-serialization）+ Ktor + kotlinx-* + Koin。其他：quickie 扫码、hiddenapibypass 预测性返回、core-splashscreen。核心：mihomo（YuKongA fork + 本地补丁）。

**依赖版本与坐标唯一来源 = `android/gradle/libs.versions.toml`**（含 `[bundles]`），mihomo 版本在 `android/gradle.properties`。应用版本、包名和 SDK 在 `android/buildSrc/src/main/kotlin/ProjectConfig.kt`；应用稳定版只修改 `VERSION_NAME`，界面只展示 `BuildConfig.VERSION_NAME`。`scripta:editor` 经 `includeBuild("../third_party/scripta")` 引入，插件由 scripta 自己的 `pluginManagement` 解析。

**Compose 稳定性**走 [compose_compiler_config.conf](android/app/compose_compiler_config.conf)，**只保留实测起作用的条目**（加之前先跑报告确认确有 unstable 参数），新增 unstable 的三方/平台字段优先进该文件而非散落 `@Stable`。三条易踩：① 条目对**子类生效**——`androidx.lifecycle.ViewModel` 一行覆盖全部 ViewModel，其内部字段稳定性因此完全不影响 composable 参数；② FQN 须与实际依赖一致，包名写错时静默失配、不报错；③ 只认整行 `//` 注释，行尾注释会被当成 matcher 内容。

**注释写什么**：只写读代码看不出来的约束与原因，如内核 / 并发时序、选用当前写法的理由。复述下一行的标签、外部参考来源（「参考 xx example」）、版本沿革（「旧版…」「不再…」）都不写。`// === X ===` 用于给 200 行以上的文件分组**多个**声明。注释若是某条不变式的唯一记录，删除前先把约束落到代码或 skill 里。

## 代码地图

分层靠**包名**，跨层即普通包引用。约定（非 Gradle 强制）：`domain.model` 只放 `@Serializable` 模型、`domain.repository` 只放仓库接口，二者不引 android/compose/ktor。

```
scripts/        prebuild.py（子模块 + 便携 JDK / Go + GeoIP）/ build.py（编译与打包）/ gen_icons.py
third_party/    mihomo（submodule，YuKongA fork branch Mishka；附加补丁在 scripts/patches/）
                scripta（includeBuild 复合构建，YAML 编辑器；app 依赖 scripta:editor）
android/        Gradle 根（wrapper + settings.gradle.kts + buildSrc + 两个模块）
                Gradle 路径仍是 :app / :baselineprofile（名字被 targetProjectPath、CI 产物路径与文档引用着）
android/app/src/main/
├── kotlin/.../stelliberty/  App / MainActivity / StellibertyApplication（startKoin + 全局初始化）
│   ├── domain/{model,repository}
│   ├── data/{api（REST/WS + MihomoConnectionManager）,bridge（StellibertyCoreBridge）,store（JsonFileStore + SubscriptionStore + ProxySelectionStore + OverrideProfileStore + RuleOverrideStore + ProfileTransformWriter）,repository（*Impl + ProfileProcessor + OverrideJsonStore + SubscriptionProxyResolver）,backup}
│   ├── platform/  service/  viewmodel/  util/  di/（4 个 Koin 模块）
│   └── ui/{navigation,component,icon（手写 MingCute 矢量图）,platform,theme,screen,util}
├── res/values{,-zh-rCN,-zh-rTW}/   assets/（构建时下载 GeoIP）
├── cpp/  process_helper.c + stelliberty_jni.c + mihomo_wrapper.c + CMakeLists.txt
├── jniLibs/arm64-v8a/libmihomo.so   native/stelliberty_core/（Go cgo 源）
└── src/release/generated/baselineProfiles/（生成产物，需提交）
```

路由清单（`ui/navigation/Route.kt`，均实现 `NavKey`）、屏幕↔ViewModel 对应、`platform`/`ui.platform`/`service` 三个包内各组件的名字与职责都能从文件名读出，此处不复述；名字不自明的那些，约束写在下方对应条目里。

## 架构

```
StellibertyApplication.startKoin ─ Koin（dataModule + androidPlatformModule + androidAppModule + viewModelModule）
  MainActivity（Koin get 取图）→ App → AppNavigation → HorizontalPager(4 Tab) + NavDisplay(二级页)
    → Screen → ViewModel → domain.repository 接口 → data.repository.*Impl
        ├→ MihomoApiClient(Ktor HTTP) + MihomoWebSocket(WS) → mihomo 进程 127.0.0.1:9090
        └→ SubscriptionStore / ProxySelectionStore（files/mihomo/ 下的 JSON 文件）
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kindness-Kismet/stelliberty_android](https://github.com/Kindness-Kismet/stelliberty_android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
