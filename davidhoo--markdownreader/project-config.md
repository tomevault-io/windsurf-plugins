---
trigger: always_on
description: > 本文件为 Codex 提供项目上下文，确保代码修改遵循项目规范。
---

# AGENTS.md — MarkdownReader 项目指南

> 本文件为 Codex 提供项目上下文，确保代码修改遵循项目规范。

## 项目简介

MarkdownReader 是一个原生 macOS Markdown 阅读器应用。不是编辑器，只是一个安静的阅读器。三栏布局：左侧目录树 + 中间渲染视图 + 右侧大纲导航。支持多窗口（每窗口独立 session）。

- **当前版本**: 2.4.7
- **最低部署**: macOS 26.0
- **Bundle ID**: `com.markdownreader.app`
- **许可证**: MIT

## 技术栈

| 组件 | 选择 |
|------|------|
| 语言 | Swift 6.2（严格并发） |
| UI 框架 | SwiftUI |
| Markdown 渲染 | cmark-gfm + WebPage（macOS 26 SwiftUI 原生 WebView） |
| 状态管理 | `@Observable`（非 ObservableObject） |
| 并发 | Swift Concurrency（async/await, `@MainActor`） |
| 构建系统 | Swift Package Manager |
| 本地化 | 自定义字典方案（L10n），非 Apple String Catalog |

## 项目结构

```
Sources/
├── MarkdownReaderKit/              # 共享库 — 渲染、主题、本地化
│   ├── Models/
│   │   ├── DisplayMode.swift              # 枚举：.rendered / .raw
│   │   ├── OutlineItem.swift              # 大纲标题（level, title, sourceLine）
│   │   ├── SourceLine.swift               # 1-based 源码行号值类型
│   │   ├── SourceScrollAnchor.swift       # 模式切换小数源码锚点 + ScrollTransfer
│   │   ├── ThemeDefinition.swift          # 5 色主题 + 33 预设 + 自定义覆盖
│   │   ├── DocumentCopyConfiguration.swift # 整篇复制配置（主应用 + Quick Look 共享）
│   │   ├── DocumentCopySymbol.swift       # 复制按钮 SF Symbol
│   │   └── DocumentCopyWebIcons.swift     # Web 侧复制图标栅格化
│   ├── Services/
│   │   ├── MarkdownHTMLService.swift      # cmark-gfm → HTML + 按需运行时资源
│   │   ├── MarkdownURLSchemeHandler.swift # URLSchemeHandler（macOS 26），mr:// 资源加载
│   │   ├── ThemeColors.swift              # 主题色彩派生（5 色 → 12+ 语义 token）
│   │   ├── OutlineService.swift           # Markdown 标题解析
│   │   ├── LanguageService.swift          # 系统语言检测
│   │   ├── LocalizationService.swift      # L10n 字典（~100 键，3 语言）
│   │   ├── EmojiService.swift             # Emoji 短码
│   │   └── SFSymbolWebImageProvider.swift # SF Symbol → Web 图标
│   └── Extensions/
│       └── ColorExtensions.swift          # Color.mix 等颜色工具
├── MarkdownReader/                 # 主应用
│   ├── App/
│   │   ├── MarkdownReaderApp.swift       # @main 入口，WindowGroup，应用级 Notification
│   │   ├── MarkdownReaderCommands.swift  # 菜单命令（FocusedValues → WindowCommandTarget）
│   │   ├── AppDelegate.swift             # NSApplicationDelegate，冷/热启动，Coordinator 持有
│   │   └── AboutWindowController.swift   # 自定义关于面板
│   ├── Models/
│   │   ├── FileNode.swift / Document.swift / FileError.swift
│   │   ├── SettingsModel.swift           # 设置单例（@Observable + UserDefaults）
│   │   ├── WindowID.swift / ResourceIdentity.swift / OpenRequest.swift / RouteDecision.swift
│   │   ├── WindowCommand.swift / CopyFeedbackState.swift
│   ├── ViewModels/
│   │   ├── WindowSession.swift           # 窗口级业务边界（持有各 VM + Undo + CommandTarget）
│   │   ├── WindowCommandTarget.swift     # 焦点窗口菜单/快捷键路由目标
│   │   ├── AppViewModel.swift            # 窗口 UI 状态（侧边栏、大纲、设置、标题）
│   │   ├── DocumentViewModel.swift       # 文档加载/保存/脏跟踪/文件监控/大纲滚动
│   │   ├── FileTreeViewModel.swift       # 目录树管理/键盘导航/文件操作
│   │   ├── FindReplaceViewModel.swift    # 查找替换
│   │   ├── CommandPaletteViewModel.swift # Cmd+P 文件搜索
│   │   └── UpdateViewModel.swift         # 自动更新检查/弹窗
│   ├── Views/
│   │   ├── WindowSceneHost.swift         # 单窗口场景根（FocusedValues 发布）
│   │   ├── WindowLifecycleBridge.swift   # NSWindow 生命周期 ↔ session
│   │   ├── ContentView.swift             # 主布局，ViewModifier 事件处理
│   │   ├── DetailView.swift              # TitleBar、模式切换锚点管线、PDF 导出
│   │   ├── WebViewMarkdownView.swift     # WebPage + WebView 渲染
│   │   ├── WebViewRenderScheduler.swift  # 渲染请求合并 / 模式过渡闸门
│   │   ├── RawMarkdownView.swift / SyntaxHighlightedEditor.swift
│   │   ├── SidebarView / OutlineView / SettingsView / FindReplaceBar / CommandPaletteView …
│   │   └── OverlayScrollerHelper.swift   # 覆盖式滚动条辅助
│   └── Services/
│       ├── WindowCoordinator.swift       # 应用级窗口注册与打开路由
│       ├── WindowRoutingEngine.swift     # 纯逻辑路由（owner → blank → create）
│       ├── ResourceIdentityService.swift # 路径标准化资源身份
│       ├── WindowUndoStore.swift         # 每窗口 per-file UndoManager
│       ├── AppStartupCoordinator.swift   # 幂等启动（预热 + 更新检查）
│       ├── WebViewWarmupService.swift    # WebPage 预热状态机
│       ├── ApplicationTerminationCoordinator.swift / UnsavedDocumentCloseCoordinator.swift
│       ├── FileService / PDFExportService / MarkdownSyntaxHighlighter / UpdateService …
│       └── MainMenuLocalizationService.swift # 主菜单本地化
└── MarkdownReaderQL/              # Quick Look 预览扩展
    └── MarkdownQLPreviewProvider.swift   # WKWebView + WKURLSchemeHandler（扩展兼容）

Tests/
└── MarkdownReaderTests/           # XCTest（~290），多窗口路由 / 渲染调度 / 源码锚点等
```

## 构建与运行

```bash
# 构建（调试）
swift build

# 构建（发布）
swift build -c release

# 运行测试
swift test

# 构建 .app 包（含签名）— arm64 only
./build-app.sh --release --sign

# 打包 DMG — arm64 only
./package.sh

# 本地发布到 GitHub（含 Homebrew Tap 同步，缺一不可）
./release-local.sh X.Y.Z
# 详见 docs/homebrew-tap-maintenance.md：正式发版后必须更新 davidhoo/homebrew-markdownreader
```

## 依赖

唯一外部依赖：**swift-markdown** 0.5.0+（Apple 官方 Markdown 解析库，基于 cmark-gfm）
- 传递依赖：swift-cmark（cmark-gfm 的 Swift 封装）
- 锁定策略：`.package(url: "https://github.com/apple/swift-markdown.git", from: "0.5.0")`

## 架构模式 (MVVM)

```
用户操作 → View → ViewModel → Service → 文件系统
                ↑
                └── State 更新 → View 刷新
```

- **ViewModels**: 全部 `@MainActor @Observable`；业务状态挂在 `WindowSession` 上，非全局单例

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidhoo/MarkdownReader](https://github.com/davidhoo/MarkdownReader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
