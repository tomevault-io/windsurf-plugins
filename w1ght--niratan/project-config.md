---
trigger: always_on
description: Niratan Mac 是 Niratan 的原生 macOS 桌面端项目。当前仓库只有一个原生 `Niratan` App target，产品能力分为小说阅读与视频学习两个模块：小说模块覆盖书架、阅读、查词、同步、制卡、本地音频、Sasayaki 和快捷键；视频模块在 Video 配置中启用本地视频库、播放器、字幕查词和视频制卡。
---

# Niratan Mac Agent 指南

Niratan Mac 是 Niratan 的原生 macOS 桌面端项目。当前仓库只有一个原生 `Niratan` App target，产品能力分为小说阅读与视频学习两个模块：小说模块覆盖书架、阅读、查词、同步、制卡、本地音频、Sasayaki 和快捷键；视频模块在 Video 配置中启用本地视频库、播放器、字幕查词和视频制卡。

本文件是所有 agent 进入仓库后的常驻规则。任务状态、执行日志、长调查过程不要写进这里。

## 工作原则

- **Mac 用户可见行为是第一真源。** 不要为了机械同步 iOS、Android 或上游实现而破坏 Mac 端已经修好的交互、排版、快捷键、同步或发布流程。
- **原生 macOS 是唯一开发和发布目标。** 小说和视频功能、修复、重构和验证只保证 `Niratan` 原生 App；不得新增非 macOS target、跨平台桥接层或替代构建路径。
- **不要用整屏重写代替原生演进。** 优先复用现有表现良好的 SwiftUI 页面和业务服务；平台差异和模块差异只在窄边界里用 AppKit / NSViewRepresentable / NSWindow 能力补齐。
- **原生 App 必须保护用户数据兼容。** 书籍目录、bookmark、sidecar、词典配置、Anki 配置、Google token 和 UserDefaults 的兼容仍是硬约束；App 启动路径不得清理旧 token 或用“首次启动清理”代替显式退出登录。
- **修 bug 不叠补丁。** 先复现、定位边界，再改最小稳定方案；Reader / WKWebView / Popup / AnkiConnect / Google Drive / Sasayaki 尤其要避免猜测式修改。
- **不回滚用户或其他 agent 的未说明改动。** 工作树可能包含未提交功能、修复或验证内容；只处理当前任务范围。
- **不擅自发版、打 tag、push 或提交。** 用户明确要求 release / commit / push 后再执行。Commit message 必须使用 Conventional Commits，例如 `feat(reader): add mouse wheel page turn`。
- 新增用户可见设置、按钮、提示、toast、alert、页面标题或 release 可见文案时，必须考虑 `Localizable.xcstrings`，至少保证中文和英文不会裸露错误文案。

## 架构基线

### 当前 App 与模块

- `Niratan`：唯一原生 macOS App target，承载小说阅读模块，并在 Video 配置中启用视频学习模块。
- Light 配置：`Niratan` scheme，使用 `Debug` / `Release`，只发布小说阅读模块，不链接、复制或运行时查找 Video/libmpv。
- Video 配置：`Niratan Video` scheme，使用 `Debug-Video` / `Release-Video` 和 `HOSHI_VIDEO`，在小说阅读模块之外启用本地视频库、播放器、字幕查词、视频挖矿和视频制卡。
- Light 和 Video 构建产物的 App 名称、bundle id `moe.shishamo.hoshi` 和持久化目录相同，可以覆盖安装并共享用户数据。
- `main`：当前发布分支。Release tag 从 `main` 打。
- `codex/` 分支：较大功能、小说/视频跨模块重构或高风险修复优先使用。

### Native 架构约束

- SwiftUI 页面能复用就复用；不要为了“原生”或“模块化”重写成熟 UI。
- 原生 macOS 最低支持版本是 macOS 26.0；新 UI 可以直接使用 macOS 26+ SwiftUI / AppKit API，不要再为 macOS 15-25 增加 material fallback，除非明确决定下调 deployment target。
- 不新增 iOS 平台条件或双平台抽象；macOS 必要能力直接使用窄范围 AppKit bridge。
- AppKit 只用于 macOS 必要能力，例如 `NSWindow`、`NSViewRepresentable`、`NSEvent`、菜单、panel、focus/key capture、文件选择、窗口 chrome。
- `NativeMac/` 承载原生 App shell、窗口呈现和验证探针，但共享业务逻辑应留在 `Core/`、`Features/`、`Models/` 等已有边界。
- 共享代码修改以原生构建和对应功能验证为准。
- Video 条件编译只能存在于功能入口和 `Features/Video/` 的依赖边界。Reader、Dictionary、Popup、LocalAudio、AnkiConnect 等共享实现不得依赖 Video 才能编译；可共享纯数据 mining metadata，以保证 Light/Video 切换时配置兼容。
- Light 配置是小说模块发布包，不得链接、复制或运行时查找 libmpv；每次修改 `Features/Video/`、构建配置或打包脚本都要同时验证 Light。

### 项目结构

- `NativeMac/`：原生 macOS App 入口、sidebar/detail、Reader/Video 窗口呈现和 AppKit 能力；当前产品主路径。
- `Core/`：核心服务与持久化，如 Anki、词典、配置、本地文件服务、查词引擎、桌面输入管理。
- `Features/Bookshelf/`：小说书架、导入、排序、同步入口。
- `Features/Reader/`：小说阅读器、Reader WebView、分页/连续阅读、统计、Sasayaki 高亮。
- `Features/Popup/`：小说和视频共享的查词弹窗、渲染 CSS/JS、单词音频、制卡入口。
- `Features/Dictionary/`：词典搜索页。
- `Features/Settings/`：设置页、外观、Anki、音频、Sasayaki、快捷键、CSS 等。
- `Features/Sync/`：Google Drive OAuth、token、同步逻辑。
- `Features/Video/`：仅 Video 配置编译的视频库、视频播放、字幕 overlay、字幕查词协调和视频制卡上下文。
- `Models/`：数据模型。
- `Util/`：工具与更新检查。
- `script/`：本地构建、验证、打包、发版脚本。
- `.github/workflows/release-mac.yml`：tag 触发 DMG 构建和 GitHub Release。

## 真源文档

- `docs/TODO.md`：短状态、下一步、阻塞项、验证入口。
- `docs/ARCHITECTURE_REFACTORING.md`：长期架构方向，不记录执行流水账。
- `docs/READER_REGRESSION_TESTING.md`：Reader 回归验证、实际 EPUB 验证矩阵和数据安全规则。
- `docs/CHANGELOG.md`：只记录用户可见变化。
- `docs/UPSTREAM_SYNC_QUEUE.md`：上游同步队列。
- `docs/AGENT_DEVELOPMENT_GUIDE.md`：当前 agent 开发规范。
- `.codex/skills/hoshi-reader-mac-workflow/SKILL.md`：本仓库任务前置工作流。

只有任务改变了对应文档的真源内容时，才更新该文档。不要把一次性调查日志、长命令输出或截图观察塞进 README 或 AGENTS。

- 任务改变 native 架构状态、小说/视频模块边界、已完成能力、剩余风险、下一步、阻塞项、验证入口或发布切换条件时，必须在同一任务内更新最小相关真源文档。
- 实现使 `docs/TODO.md`、`docs/ARCHITECTURE_REFACTORING.md` 或 `docs/READER_REGRESSION_TESTING.md` 的现状描述失真时，不得只改代码；声明完成前必须同步文档。
- native 架构或模块实现引起的真源文档更新默认放在同一个 commit，除非用户明确要求拆分。不要单独制造没有状态变化的文档流水账 commit。

## 经验沉淀

- 如果 agent 犯错后定位到未来可能复发的问题，应把最小可执行规则沉淀到对应真源文档。
- 需要所有会话常驻的仓库级规则才写入 `AGENTS.md`。
- 验证矩阵和脚本入口写入 `docs/READER_REGRESSION_TESTING.md` 或 `docs/TODO.md`。
- 当前架构事实和长期演进方向写入 `docs/ARCHITECTURE_REFACTORING.md` 或 `docs/TODO.md`。
- 沉淀内容必须具体、可执行、低歧义；先查是否已有等价规则，有则更新原规则。

## 构建与启动

默认构建和验证原生 macOS App 与两个发布包：

```bash
./script/build_and_run.sh
./script/build_and_run.sh --verify
./script/build_and_run.sh --video
./script/build_and_run.sh --video --verify
```

`script/build_and_run_native.sh` 是同一原生 target 的显式入口。普通签名构建可能因为本机缺少 `Mac Development` 证书失败；除非任务是签名/发布，不要把证书错误当作代码回归。

构建、启动和 UI 验证必须确认实际 App 身份：当前 Light/Video 产物的 bundle id 都是 `moe.shishamo.hoshi`。`--verify` 必须同时校验构建产物的 `CFBundleIdentifier` 和运行中进程的完整 executable 路径；仅凭进程名、窗口标题或 `/Applications/Niratan.app` 不得宣称验证成功。Computer Use 等 GUI 工具必须传入本次 DerivedData 产物的绝对 `.app` 路径或唯一 bundle id，不得只传模糊名称 `Niratan`，以免启动旧安装包。
多个 Codex 会话并行做 App UI 验证时，每个会话必须使用不同 `./script/build_and_run.sh --instance <id>` 或显式 `HOSHI_DERIVED_DATA_PATH`，并只操作该命令输出的 `.app` / executable path；`--instance` 只隔离构建产物、启动清理、进程验证和日志，不隔离同一 bundle id 下的 UserDefaults、Application Support、书籍 sidecar 或 Sasayaki 播放数据。

Video 配置通过 `./script/build_and_run.sh --video` 启动，内部使用 `Niratan Video` scheme，但构建产物仍是同 bundle id 的 `Niratan.app`。首次构建前运行 `./script/bootstrap_libmpv.sh`；依赖只允许落在被忽略的 `Vendor/libmpv/include/mpv/` 和 `Vendor/libmpv/lib/`。

## Release 流程

- 版本号来自 `Niratan.xcodeproj/project.pbxproj` 的 `MARKETING_VERSION`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [W1ght/Niratan](https://github.com/W1ght/Niratan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
