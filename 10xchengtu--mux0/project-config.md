---
trigger: always_on
description: macOS 标签页 + 分割窗格式终端 app，以 libghostty 作为终端引擎，Swift/AppKit + SwiftUI 实现。
---

# mux0

macOS 标签页 + 分割窗格式终端 app，以 libghostty 作为终端引擎，Swift/AppKit + SwiftUI 实现。

## Quick Start

```bash
# 1. 首次构建 libghostty（只需一次）
./scripts/build-vendor.sh

# 2. 生成 Xcode 工程
xcodegen generate

# 3. 构建并运行（命令行）
xcodebuild -project mux0.xcodeproj -scheme mux0 -configuration Debug build

# 4. 运行测试
xcodebuild test -project mux0.xcodeproj -scheme mux0Tests

# 5. 检查文档/目录是否漂移
./scripts/check-doc-drift.sh
```

## Architecture Overview

SwiftUI 侧边栏 + AppKit 标签页/分割窗格混合架构。每个 Workspace 持有一组 TerminalTab，每个 Tab 存一棵 `SplitNode` 树（二叉 split + terminal 叶子）。WorkspaceStore (@Observable) 是唯一状态来源，通过 environment 注入到所有视图。libghostty C API 封装在 GhosttyBridge 单例中，不直接调用 `ghostty_*` 函数在其他文件。

详见 `docs/architecture.md`。

> **历史说明**：早期版本是"白板/Canvas 无限画布"形态，2026-04 迁移到标签页 + 分割。相关决策见 `docs/decisions/003-tabs-over-canvas.md`，旧的设计笔记保留在 `mux0/Canvas/DESIGN_NOTES.md` 仅作参考。

## Directory Structure

```
mux0/
├── mux0App.swift          — @main 入口，ghostty init，全局 menu commands
├── ContentView.swift      — 根 ZStack/HStack：SidebarView + TabBridge（+ SettingsView 叠层）
├── WindowAccessor.swift   — NSViewRepresentable，把 NSWindow 交给 ghostty 安装 blur layer
├── Bridge/
│   ├── TabBridge.swift          — TabContentView SwiftUI ↔ AppKit 桥接
│   └── SidebarListBridge.swift  — WorkspaceListView SwiftUI ↔ AppKit 桥接
├── Canvas/
│   └── DESIGN_NOTES.md    — 仅保留旧白板形态的设计笔记（代码已全部删除）
├── TabContent/
│   ├── TabBarView.swift       — NSView，水平标签条（新建/关闭/重命名/拖拽）
│   ├── TabContentView.swift   — NSView，承载当前 workspace 的 tabs，管理 SplitPaneView 缓存
│   ├── SplitPaneView.swift    — 递归 NSSplitView 渲染 SplitNode，drag divider + 键盘焦点导航
│   ├── SurfaceScrollView.swift — NSScrollView 包装 GhosttyTerminalView，提供原生 overlay 滚动条（消费 ghostty SCROLLBAR/CELL_SIZE action，拖拽回写 `scroll_to_row:N`）
│   └── PasteboardTypes.swift  — .mux0Tab UTI 常量
├── Ghostty/
│   ├── ghostty-bridging-header.h  — 桥接头文件
│   ├── GhosttyBridge.swift        — libghostty 单例（app/config 生命周期 + window blur）
│   └── GhosttyTerminalView.swift  — NSView，持有 ghostty_surface_t，Metal 渲染
├── Localization/
│   ├── LanguageStore.swift  — @Observable，语言偏好 + UserDefaults 持久化 + effectiveBundle + locale + tick
│   ├── L10n.swift           — 常量命名空间（LocalizedStringResource + AppKit helper L10n.string(_:args...)）
│   └── Localizable.xcstrings    — Xcode String Catalog（en source + zh-Hans 翻译）
├── Metadata/
│   ├── WorkspaceMetadata.swift  — git/PR/通知 数据结构
│   └── MetadataRefresher.swift  — 5s 轮询 + OSC 通知处理 + onRefresh 回调
├── Models/
│   ├── Workspace.swift            — Workspace / TerminalTab / SplitNode / SplitDirection（Codable）
│   ├── WorkspaceStore.swift       — @Observable，workspace / tab / split CRUD + UserDefaults 持久化
│   ├── TerminalStatus.swift       — 单终端运行状态枚举（running / idle / needsInput）
│   ├── TerminalStatusStore.swift  — @Observable，终端状态聚合（供 sidebar / tab 图标读取）
│   ├── TerminalPwdStore.swift     — @Observable，terminalId → pwd 映射（ghostty PWD action 喂入，sidebar git 分支读取）
│   ├── HookDispatcher.swift       — 按 agent toggle 过滤 hook 事件 + 派生 status icon UI 主开关
│   ├── HookMessage.swift          — agent/shell hook 的 JSON wire format
│   └── HookSocketListener.swift   — Unix domain socket 监听，解析 HookMessage → statusStore
├── Settings/
│   ├── SettingsView.swift         — SwiftUI 根面板，承载六个 Section
│   ├── SettingsConfigStore.swift  — @Observable，读写 mux0 config（200ms debounce）
│   ├── SettingsTabBarView.swift   — Appearance / Font / Terminal / Shell / Agents / Update 分类切换
│   ├── SettingsSection.swift      — Section 枚举定义
│   ├── ThemeCatalog.swift         — 只读扫描 bundle 内 ghostty/themes/
│   ├── Components/                — 复用 SwiftUI 控件（BoundControls / FontPicker / ThemePicker / ...）
│   └── Sections/                  — 六个 Section 的视图实现（Appearance / Font / Terminal / Shell / Agents / Update）
├── Sidebar/
│   ├── SidebarView.swift              — SwiftUI 壳：header / footer / alert / 通知订阅 / refresher 生命周期
│   ├── WorkspaceListView.swift        — AppKit 列表 + private WorkspaceRowItemView
│   └── WorkspacePasteboardType.swift  — .mux0Workspace UTI 常量
├── Update/
│   ├── UpdateState.swift         — 自动更新 UI 状态枚举（idle / checking / upToDate / updateAvailable / downloading / readyToInstall / error）
│   ├── UpdateStore.swift         — @Observable，自动更新状态存储（sidebar 红点 + Settings Update section 都读这个）
│   ├── SparkleBridge.swift       — 单例，持有 SPUUpdater，对外暴露 checkForUpdates/downloadAndInstall/skip/dismiss/retry；Debug 下为空 stub
│   └── UpdateUserDriver.swift    — 实现 Sparkle 的 SPUUserDriver，事件 → UpdateStore 变更（MainActor，Release-only）
└── Theme/
    ├── AppTheme.swift               — 设计 token 结构体（sidebar / canvas / border / accent / ...）
    ├── DesignTokens.swift           — token 定义与 spacing / radius / layout 常量
    ├── GhosttyConfigReader.swift    — 从 ghostty config 读颜色
    ├── ThemeManager.swift           — @Observable，主题解析与分发（含 opacity / blur）
    ├── IconButton.swift             — 紧凑图标按钮（hover/pressed 走 theme token）
    └── TerminalStatusIconView.swift — 终端状态圆点图标
```

## Key Conventions

1. **所有颜色取自 AppTheme token** — 禁止在视图里硬编码 `Color(...)` 或 `NSColor(...)`，从 `@Environment(ThemeManager.self)` 取 `.currentTheme` 或直接传 `theme` 参数
2. **ghostty API 只在 GhosttyBridge 和 GhosttyTerminalView 中调用** — 其他文件不 `import` 也不直接调用 `ghostty_*` 函数

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [10xChengTu/MUX0](https://github.com/10xChengTu/MUX0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
