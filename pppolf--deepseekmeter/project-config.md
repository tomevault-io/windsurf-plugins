---
trigger: always_on
description: > This file is the operating manual for AI coding agents (Claude Code, Cursor, Codex, etc.).
---

# AGENTS.md — DeepSeekMeter 仓库操作规范

> This file is the operating manual for AI coding agents (Claude Code, Cursor, Codex, etc.).
> 开始在本仓库修改代码之前，请先完整阅读本文件，并严格遵守其中的规范与边界。

## 1. 项目是什么

DeepSeekMeter 是一个 **macOS 菜单栏小工具**（SwiftUI + AppKit）：实时展示 DeepSeek 平台账户的余额、本月费用、Token 用量与按天趋势。

- 工具链：Swift 6（swift-tools-version 6.0），**Swift 5 语言模式**（Package.swift 中显式设置）
- 平台：macOS 14+（Apple Silicon / Intel 均可），仅 macOS，无 iOS/iPadOS
- 移动端：iOS 版（`ios/`，开发中）与 Android 版（规划中），总体方案见 MOBILE-PLAN.md
- 构建：Swift Package Manager，**无 Xcode 工程、无任何第三方依赖、单 target**（仅指 macOS 包；iOS 工程在 `ios/` 内，见第 8 节红线 13）
- 测试：轻量自测（swiftc 直接编译运行，**不依赖 XCTest**）
- CI：GitHub Actions（push main / PR 触发）；发布：打 `v*` 标签自动出 DMG Release
- 用户文档：README.md（英文）+ README.zh-CN.md（中文），双语惯例

## 2. 常用命令（改动后必须本地验证）

```bash
swift run                        # 开发模式运行（无 .app 外壳）
swift build                      # debug 构建
swift build -c release           # release 构建
bash Scripts/run-tests.sh        # 运行自测（全绿才算通过）
bash Scripts/build-app.sh release # 组装 build/DeepSeekMeter.app 并签名
bash Scripts/install.sh          # 构建 + 安装到 /Applications 并启动

# iOS 版验证（核心包自测 + 工程结构静态校验必跑；有 Xcode 时还会构建 App 冒烟）
bash Scripts/run-ios-tests.sh
python3 Scripts/check-ios-project.py   # 单独跑工程结构校验（无 Xcode 环境的把关）

# 本机有 Xcode 后：一键跑 iOS 模拟器（构建 + 运行 + 截图），详见 docs/install-xcode-and-run.md
bash Scripts/run-ios-simulator.sh

# Android 核心单测（需 JDK 17 + Android SDK；JVM 直跑无需设备）
cd android && ./gradlew :core:test
```

CI 的验证链：`swift build` → `swift build -c release` → `run-tests.sh` → `build-app.sh release` → 冒烟启动 6 秒。
**任何改动都必须保证这条链全部通过。**

注意：`swift run` 时 `Bundle.main.bundleIdentifier` 为 nil，开机自启注册会被跳过（SettingsStore 中已处理），这是预期行为，不是 Bug。

## 3. 仓库结构

```
Sources/DeepSeekMeter/
  AppMain.swift                  @main 入口；.accessory 激活策略（仅菜单栏，无 Dock 图标）
  AppDelegate.swift              生命周期组装：SettingsStore → AppModel → StatusItemController
  StatusItemController.swift     菜单栏 NSStatusItem + NSPopover 宿主（内容尺寸自适应）
  AppModel.swift                 状态中枢：@MainActor ObservableObject；轮询、拉取、错误状态
  PlatformService.swift          DeepSeek 平台私有接口客户端 + PlatformError
  Models.swift                   网络模型（Decodable）+ MonthUsage 聚合模型
  SettingsStore.swift            设置持久化（UserDefaults）+ 开机自启 + 旧钥匙串一次性迁移
  Formatting.swift               format() / currencySymbol() 等纯函数
  LoginWindowController.swift    内嵌官方登录页（WKWebView）+ Token 自动提取
  Views/
    PopoverView.swift            悬浮窗主界面（SwiftUI）
    SparklineView.swift          Token 按天用量柱状图
windows/                         Windows 版（.NET 8 + WPF，与 macOS 版功能对齐）
  src/DeepSeekMeter.Core/        纯逻辑库：Models / Formatting / PlatformService / SettingsStore
  src/DeepSeekMeter/             WPF 应用：MainViewModel / TrayIconController / PopoverWindow / LoginWindow 等
  tests/DeepSeekMeter.Selftest/  轻量自测（控制台，零测试框架）
  README.md                      Windows 版说明（含与 macOS 版对应关系）
ios/                             iOS 版（开发中，详见 MOBILE-PLAN.md）
  DeepSeekMeterCore/             共享核心 Swift Package（PlatformService / Models / Formatting / TokenStoring / AppModel，零第三方依赖；AppModel 注入 TokenStoring+URLSession，可在 macOS 上自测）
  DeepSeekMeter.xcodeproj        iOS App 工程（SwiftUI，Xcode 16 同步文件夹格式）
  DeepSeekMeter/                 iOS App 源码（AppMain / TokenStore(Keychain) / Views/ / NotificationService / BackgroundRefreshService）
  DeepSeekMeterWidget/            WidgetKit 余额小组件（快照驱动；Token 不进 App Group 共享容器）
  DeepSeekMeterCore/Sources/DeepSeekMeterCoreSelftest/  核心轻量自测（swift run 直接跑，不依赖 XCTest）
android/                         Android 版（开发中，详见 MOBILE-PLAN.md 第 4 节）
  core/                          纯逻辑核心模块（Kotlin，零第三方业务依赖；org.json 手写映射）
  core/src/test/                 本地 JVM 单测（无需设备；org.json 测试期用官方 jar 替身）
Scripts/
  build-app.sh / install.sh / notarize.sh / run-tests.sh / run-ios-tests.sh / make-icon.sh / generate-icon.swift
  Info.plist                     应用包信息（**版本号在这里改**）
  selftest/main.swift            轻量自测源码（swiftc 编译运行）
.github/workflows/
  ci.yml                         push main / PR：macOS（构建+自测+打包+冒烟）+ Windows（构建+自测+冒烟）
  release.yml                    打 v* 标签：构建 DMG 并发布 GitHub Release
```

## 4. 架构分层（改动必须遵循）

依赖方向只允许自上而下：

```
UI（Views / StatusItemController / LoginWindowController）
        ↓ 读写 @Published 状态
AppModel / SettingsStore（@MainActor，状态与持久化）
        ↓ 调用
PlatformService（网络唯一入口） + Models（解码模型）
        ↓
Foundation / AppKit / SwiftUI / WebKit
```

- 所有网络请求只能经过 `PlatformService`；错误统一转为 `PlatformError`（带用户可读的中文 message）
- UI 只读模型层的 `@Published` 状态，**不直接发起网络请求**
- 新接口的响应模型写成 `Decodable` struct 放进 Models.swift，沿用平台 `{code, msg, data: {biz_code, biz_msg, biz_data}}` 包裹结构（注意 `biz_data` 有时是对象、有时是数组，以真实响应为准）
- 纯函数（格式化、币种符号、聚合计算）放 Formatting.swift / Models.swift 计算属性，并补自测
- 代码注释用中文 `///`；分组用 `// MARK: -`；UI 文案用中文

## 5. 提交规范（Conventional Commits，中文描述）

```
<type>(<scope>): <中文描述>
```

- type：`feat` / `fix` / `docs` / `refactor` / `chore` / `ci` / `test` / `perf`
- scope 可选（如 `ci`、`ui`、`api`）；描述用中文，一句话说清改了什么、为什么
- 示例：`fix(ci): build-app.sh 在无开发者证书环境下不再被 set -e 中断`

要求：
- 小步提交，一次提交只做一件事
- 不提交构建产物：`.build/`、`build/`、`.DS_Store` 已被 .gitignore 忽略，不要 `git add -f` 强加
- 提交信息与 diff 中不得出现真实 Token / 凭据

## 6. 分支与 PR 流程

1. 从 `main` 切分支，命名建议 `feat/xxx`、`fix/xxx`
2. 本地验证完整链（见第 2 节）
3. 提 PR 到 `main`，按 .github/pull_request_template.md 逐项填写勾选
4. PR 触发 CI，**CI 全绿才可合入**（建议 squash 合并）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pppolf/DeepSeekMeter](https://github.com/pppolf/DeepSeekMeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
