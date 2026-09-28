---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Piko 是 PikPak 的第三方跨平台客户端。Android 与 Windows 共用一套 Material 3 Expressive 界面，
按窗口宽度自适应；业务逻辑与屏幕状态在 `shared`，界面在 `ui`，两端只剩入口与平台实现。

## 常用命令

```bash
./gradlew :app:compileDebugKotlin          # Android 编译（最快的语法与类型检查）
./gradlew :desktopApp:compileKotlinDesktop # Desktop 编译，注意不是 compileKotlinJvm
./gradlew :app:installDebug                # 装到已连接设备，包名 dev.piko.debug
./gradlew :desktopApp:run                  # 跑 Windows 桌面端
./gradlew :app:testDebugUnitTest           # 单元测试，目前只有 app/src/test
./gradlew :app:testDebugUnitTest --tests '*FileNameSanitizerTest*'   # 跑单个测试
```

改完务必两端都编译：`shared` 与 `ui` 的改动会同时波及 `app` 与 `desktopApp`，只编译一端看不出来。
`ui` 的桌面端与 Android 端用的 material3 版本不同（见「桌面端」一节），同一行代码可能只在一端报错。

版本号来自环境变量 `PIKO_VERSION_NAME` / `PIKO_VERSION_CODE`，本地不设则用默认值，无需配置。

## 与 pikpak-kotlin SDK 的关系

网盘能力全部来自 `io.github.nihildigit:pikpak-kotlin`（版本在 `gradle/libs.versions.toml`），
作者同一人，源码通常在本机 `../pikpak-kotlin`。

**需要改 SDK 时**：在 SDK 仓库 `./gradlew publishToMavenLocal`（那边需要 `ANDROID_HOME`，
仓库里没有 `local.properties`），把版本指向对应的 SNAPSHOT，并在 `settings.gradle.kts` 的
`dependencyResolutionManagement` 里临时加 `mavenLocal`。**提交前必须移除 mavenLocal 并指向
已发布版本**——发版在干净 runner 上构建，本机 `~/.m2` 在那里不存在，否则 release 必挂。

## 发版

两个仓库都由 `v*` tag 触发 GitHub Actions。piko 依赖已发布的 SDK，所以顺序不能颠倒：
先发 SDK（tag → Maven Central，`automaticRelease = true` 无需手动确认，同步到 repo1 约数分钟），
确认 `repo1.maven.org` 能解析到新版本后，再改 piko 的依赖版本、提交、打 tag。

自有仓库不走 PR，直接在 `main` 上提交。

Release 正文由 `release.yml` 按 `.github/release-notes.md` 生成：`## 下载` 起是按设备列出的附件表与校验说明。
**更新日志发版后手写，放在正文最前面、`## 下载` 之前**：应用内
更新弹窗读到这个标题就截断（`GithubReleases.kt` 的 `updateNotesOf`），标题改动要两边一起改，`ReleaseNotesTest` 会报错。
`gh release edit --notes-file` 替换整段正文而不是追加，改之前先用 `gh release view <tag> --json body` 读回原文，
把更新日志拼在前面再写回，否则附件表就丢了。

更新日志写给下载的人看，照 Bilby 的格式：一行概述，然后 `## 修复` 与 `## 变化`，每条一句书面语，写读者能察觉的
现象或行为变化，不写文件名、类型名与提交标题，读者看不到的重构不写。`## 修复` 只列已发布版本里存在的问题：
本次新增的功能在发布前出过又修掉的问题，读者从未遇到，不列。写之前先读上一个版本的正文，保持一致。

## 架构

四个模块：`shared`（状态与业务，commonMain + android/desktop 两个 target）、`ui`（共享界面，
同样两个 target）、`app`（Android 入口、平台实现与播放器）、`desktopApp`（Windows 入口、
平台实现与播放器窗口）。

### 界面写一次

`ui/src/commonMain` 是全部界面：主题、导航、网盘、传输、设置、回收站、登录与各组件，两端共用。
入口是 `PikoApp`：`MainActivity` 与桌面的 `Main.kt` 各自拼好 `PikoServices`（进程级的仓库与调度器）
与 `PikoPlatform`（平台能力），传进去即可。屏幕里经 `LocalPikoServices`、`LocalPikoPlatform` 取用，
不要再引用 `PikoApplication.instance`。

`shared/.../shared/state/DriveScreenState.kt` 仍是核心接缝：文件列表、加载态、排序、搜索、多选、
启发式折叠、防窥揭示、目录导航与增删改动作都在这里。约定：
- 状态用 Compose 的 `State` 而非 `StateFlow`；为此 `shared` 对 compose runtime 用的是 `api`。
- 派生值用 `derivedStateOf`，不要写成 getter——它们每帧会被读到多次。
- 面向用户的提示走 `messages: SharedFlow<String>` 事件流，界面用 Snackbar 呈现。用状态表达会在重组时重放。
- 长驻的错误态（如 `loadError`）才用状态，它描述的是「眼前这份数据是旧的」。

新增屏幕状态时照这个形状做。已下沉的 state holder 都在 `shared/.../shared/state/`：
`DriveScreenState`、`InstantSheetState`（秒传与磁力解析，多条链接时由 `InstantBatchState` 为每条各建一个）、
`OfflineTasksState`（云端离线任务，轮询由调用方的协程控制启停）、`TrashScreenState`、`LoginState`、
`FolderPickerState`（自带路径栈）、`DuplicateFinderState`（查重）、`ArchiveExtractSession`（服务端解压，
进程级，挂在 `PikoServices` 上，离开网盘页照常进行）。
播放器的准备策略是 `shared/.../shared/media/player/PlayerScreenState`，见「播放器」一节。

### 响应式布局

布局只看窗口宽度，不看设备：`ui/.../adaptive/WindowWidth.kt` 按 M3 断点给出 compact、medium、
expanded。桌面窗口缩放与平板分屏走同一套判断，桌面体验以 Android 平板为准。
- 导航：`NavigationSuiteScaffold` 在 compact 下是底部导航栏，更宽时换成侧边导航栏。
- 回收站：compact 下是盖住整窗的压栈页；medium 在导航栏右侧的内容区里；expanded 与「我的」并排成两栏。
- 行长：设置、传输、回收站的行内容收在 840dp 以内居中。列表本身仍铺满窗口（用 `readableSidePadding`
  算 contentPadding），两侧空白处滚轮也能滚。
- 对话框：目录选择器在 compact 下全屏，更宽时是居中的基本对话框。

鼠标与键盘：条目右键弹出与操作面板相同的菜单（`ContextMenuArea`，动作列表 `fileActions` 两处共用）；
图标按钮用 `TooltipIconButton`，快捷键写在提示里；Esc 经 `BackHandler` 触发返回；网盘页快捷键见
`DriveScreen` 的 `handleShortcut`。新加的界面同时照顾触屏与鼠标：下拉刷新之类只有触屏能用的操作，
宽窗口要另给按钮。

### 平台差异用接口，不用 expect/actual

`PikoUserPreferences`、`PikoSessionStore`、`PikoDownloadStorage`、`PikoSegmentDownloader`
都是 commonMain 的接口，Android 与 Desktop 各有实现。界面要的平台能力（剪贴板、系统取色、
下载位置选择、本地文件的打开与分享、片段预览的播放后端、全屏对话框、应用内更新）集中在
`ui/.../platform/PikoPlatform.kt`，实现是 `AndroidPikoPlatform` 与 `DesktopPikoPlatform`。
平台没有的能力返回 null 或 false，界面据此隐藏入口，例如桌面端没有系统分享。应用内更新两端都有，
检查与版本比较在 `shared/.../shared/update`，安装各走各的：Android 交给 PackageInstaller，桌面端按文件清单
决定只换 jar、AOT 缓存等五个文件还是整包 MSI，由 `apply-update.ps1` 在应用退出后执行。

本机文件上传的调度在 `shared/.../shared/upload/PikoUploadCoordinator`，一次传一个，会话随任务存盘以便跨进程续传；
平台只提供读文件（`PikoUploadSources`，桌面端是路径，Android 是 content: URI）与选择器（`PikoPlatform.uploadPicker`）。

**加一个偏好项要同时改三处**：接口、
`SessionManager`（Android，DataStore）、`DesktopPikoPreferences`（Desktop，`DesktopSettingsStore`）。

### 全局导航栈在仓库层

`PikoDriveRepository` 持有 `folderStackFlow`，是网盘主界面的全局位置，并持久化。目录选择器
一类的浮层**必须维护自己的路径栈**，碰它会把主界面的位置一起改掉。
仓库层还有 `refreshEvents`，供界面外的改动（如回收站恢复）通知列表刷新，`DriveScreenState`
已在 `init` 里订阅，视图不要再订阅一遍。

## PikPak API 的既有约束

这些是实测结论，不要重新推导：

- **没有服务端按名搜索**。`/drive/v1/files` 的 `filters` 只认 phase / trashed / kind /
  starred / modified_time，`name` 一律 404，`q` 与 `search_text` 被接受后忽略。官方 Web 端
  自己也是本地过滤。全盘搜索只能是客户端递归遍历（SDK 的 `searchFilesRecursive`）。
- **离线任务只吃整条磁力 URL**，`createUrlFile` 没有按文件选择的参数，`ResolvedFile` 也不带
  文件索引。所以「秒传一部分、离线另一部分」必然产生重复文件。
- **gcid 是内容哈希**，与文件名无关。已在网盘的文件其 gcid 就在 `FileStat.hash` 里。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NihilDigit/piko](https://github.com/NihilDigit/piko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
