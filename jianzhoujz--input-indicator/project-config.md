---
trigger: always_on
description: 本文件是项目唯一的「面向 AI 阅读」的入口文档。任何 AI Agent（Claude
---

# AGENTS.md — 给 AI 协作 Agent 的项目指南

本文件是项目唯一的「面向 AI 阅读」的入口文档。任何 AI Agent（Claude
Code / Cursor / Codex / Gemini Code Assist / Cline 等）在开始工作前
都应先读完本文，再去翻代码。本文回答四个问题：

1. **这是什么项目？** —— 一句话定位 + 用户视角
2. **代码长什么样？** —— 单文件 Swift 应用的结构地图
3. **状态推断怎么做？为什么这么做？** —— 核心算法与设计原则
4. **开发与发布流程？** —— 构建、安装、调试、提交规范

---

## 1. 项目一句话定位

豆包输入法 / 微信输入法的 macOS 菜单栏中英文状态指示器。**单文件
Swift 应用**（[Sources/DoubaoInputIndicator.swift](Sources/DoubaoInputIndicator.swift)，约 1700
行），通过编译期 `-D WETYPE` flag 切出两个产品（豆包版 / 微信版）。

### 用户视角

菜单栏出现一个 emoji 图标，指示当前 IME 的中英文子状态：

| 图标 | 含义 |
|---|---|
| `🇨🇳` | 中文模式 |
| `🇺🇸` | 英文模式 |
| `🫥` | 状态未知，需要校准或等待自动校准 |
| `🤐` | 当前不是目标输入法（豆包/微信），不显示状态 |
| `🥶` | 输入监控权限未开启 |

`LSUIElement` accessory 应用——无 Dock 图标、无主窗口，常驻菜单栏。

### 为什么需要这个东西

豆包、微信输入法的「中文 / 英文」子状态保存在它们自己的进程内存里，
**没有任何官方 API 能从外部读到**（详见 §3.4 的调研结论）。系统输入
法菜单只能显示「当前选中了哪个输入法」，无法显示该输入法内部的中英
状态。因此本项目用多种间接信号（Shift 键、候选词窗、AX 模式指示框）
推断状态。

---

## 2. 代码结构地图

### 仓库布局

```
input-indicator/
├── AGENTS.md                       ← 本文件，AI 入口
├── README.md                       ← 用户视角的安装与使用文档
├── DEVELOPMENT_CONTEXT.md          ← 仅本地，作者私人开发笔记（.gitignore）
├── Sources/
│   └── DoubaoInputIndicator.swift  ← 全部业务代码，单文件
├── Resources/
│   └── AppIcon.icns                ← 应用图标
├── tools/
│   └── make_app_icon.swift         ← 从 emoji 生成 iconset 的脚本
├── packaging/
│   └── dmg/                        ← DMG 打包用的 .DS_Store / 背景图
├── scripts/
│   └── package-release.sh          ← 同时打两个 DMG 给 Homebrew tap 用
├── docs/
│   └── images/                     ← README 用图
├── build.sh                        ← swiftc + lipo + codesign，输出 build/<App>.app
├── install.sh                      ← 构建 + 装到 /Applications + LaunchAgent
├── uninstall.sh                    ← 反向操作
└── package-dmg.sh                  ← 从 build/<App>.app 打 dmg 到 dist/
```

### 单文件源码结构（[Sources/DoubaoInputIndicator.swift](Sources/DoubaoInputIndicator.swift)）

按出现顺序：

| 行号 | 符号 | 职责 |
|---|---|---|
| [L7-L38](Sources/DoubaoInputIndicator.swift#L7) | `enum DisplayMode` | 状态枚举（chinese / english / unknown / nonTarget） + 显示用的 emoji 标题 |
| [L40-L47](Sources/DoubaoInputIndicator.swift#L40) | `struct AppConfig` | 变体配置：app 名、目标 IME bundle ID、UserDefaults key、日志文件名 |
| [L49-L82](Sources/DoubaoInputIndicator.swift#L49) | GitHub release 工具 | `VersionNumber` 比较 + 检查更新所需的 URL 常量 |
| [L84-L102](Sources/DoubaoInputIndicator.swift#L84) | `#if WETYPE` 分支 | **唯一**编译期分歧：豆包 vs 微信的 `appConfig` 实例 |
| [L104-L131](Sources/DoubaoInputIndicator.swift#L104) | `class InputSourceReader` | `TISCopyCurrentKeyboardInputSource` 的封装，返回当前输入源标识 |
| [L133-L347](Sources/DoubaoInputIndicator.swift#L133) | `class CandidateWindowMonitor` | 候选窗 / 模式指示器小窗口的扫描与 AX 文本读取（**核心算法在这里**） |
| [L349-L1694](Sources/DoubaoInputIndicator.swift#L349) | `class AppDelegate` | 状态机、事件监听、菜单构建、权限管理、日志、自启、更新检查 |
| [L1696-L1699](Sources/DoubaoInputIndicator.swift#L1696) | 入口 | `NSApplication.shared.run()` |

### `CandidateWindowMonitor` 关键方法

| 行号 | 方法 | 用途 |
|---|---|---|
| [L189-L247](Sources/DoubaoInputIndicator.swift#L189) | `snapshot(bundleID:indicatorMinSize:indicatorMaxSize:logHandler:)` | 一次扫描，返回候选窗是否可见 + 所有指示器小窗口的 ID 与屏幕矩形 |
| [L256-L274](Sources/DoubaoInputIndicator.swift#L256) | `cachedFindIMEProcessID(_:)` | PID 5 秒缓存，避免每次都扫 `runningApplications` |
| [L292-L315](Sources/DoubaoInputIndicator.swift#L292) | `recognizeModeFromIndicatorRect(pid:rect:)` | **窄域 AX 读取**：只读指示器矩形上的 AX 子树，严格匹配单字符「中」「英」 |
| [L319-L345](Sources/DoubaoInputIndicator.swift#L319) | `collectTexts(from:into:depth:maxDepth:childCap:)` | 受限的 AX 子树文本收集，深度 ≤3、每层 ≤8 子节点 |

### `AppDelegate` 关键方法

| 行号 | 方法 | 用途 |
|---|---|---|
| [L431-L458](Sources/DoubaoInputIndicator.swift#L431) | `applicationDidFinishLaunching` | 装事件 tap、装权限、起 0.3s 主轮询 timer |
| [L506-L554](Sources/DoubaoInputIndicator.swift#L506) | `installEventTap` | 装 CGEvent listen-only tap（要 Input Monitoring 权限） |
| [L568-L582](Sources/DoubaoInputIndicator.swift#L568) | `installGlobalMonitor` | NSEvent 全局监听 fallback |
| [L591-L646](Sources/DoubaoInputIndicator.swift#L591) | `handle(event:)` / `handle(type:event:)` | 把 NSEvent / CGEvent 路由到 keyDown / mouseDown / flagsChanged |
| [L653-L725](Sources/DoubaoInputIndicator.swift#L653) | `pollCandidateWindow` | **每 0.3s 调一次**，触发指示器窗 AX 读取与候选窗校准 |
| [L768-L811](Sources/DoubaoInputIndicator.swift#L768) | `noteAlphaKeyDown` | 累计 alpha 键计数，安排 0.25s 后的延迟候选窗检查 |
| [L814-L854](Sources/DoubaoInputIndicator.swift#L814) | `performCandidateWindowCheck` | 候选窗看到 → 中文。**注意：候选窗没看到不会推断英文**（见 §3） |
| [L1010-L1156](Sources/DoubaoInputIndicator.swift#L1010) | `handleFlagsChanged` / `handleShiftKeyChange` / `finishShiftTap` | Shift 翻转的完整状态机 |
| [L1145-L1156](Sources/DoubaoInputIndicator.swift#L1145) | `schedulePostShiftVerification` | Shift 后 0.15 / 0.4 / 0.7 秒各跑一次 `pollCandidateWindow`，捕获指示器窗 |
| [L1176-L1190](Sources/DoubaoInputIndicator.swift#L1176) | `markTargetModeUnknown` | 失去信心时清空状态，显示 🫥 |
| [L1236-L1351](Sources/DoubaoInputIndicator.swift#L1236) | `rebuildMenu` | 菜单完整重建（每次状态变化或菜单将打开时） |

### 两个变体的差异点

整个项目只有 [L84-L102](Sources/DoubaoInputIndicator.swift#L84) 一处 `#if WETYPE` 分支：

| 字段 | 豆包 | 微信 |
|---|---|---|
| `appName` | `DoubaoInputIndicator` | `WeTypeInputIndicator` |
| `displayName` | `豆包输入法指示器` | `微信输入法指示器` |
| `targetInputMethodBundleID` | `com.bytedance.inputmethod.doubaoime` | `com.tencent.inputmethod.wetype` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianzhoujz/input-indicator](https://github.com/jianzhoujz/input-indicator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
