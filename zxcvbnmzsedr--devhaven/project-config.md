---
trigger: always_on
description: DevHaven 当前仓库已经收口为 **纯 macOS 原生主线**：唯一保留的应用源码位于 `macos/`，技术栈为 **SwiftUI + AppKit + Swift Package + GhosttyKit + Sparkle**。
---

# 项目概览（DevHaven）

DevHaven 当前仓库已经收口为 **纯 macOS 原生主线**：唯一保留的应用源码位于 `macos/`，技术栈为 **SwiftUI + AppKit + Swift Package + GhosttyKit + Sparkle**。

## 复杂布局实现约定

- 保持 **SwiftUI + AppKit** 作为应用主壳、窗口生命周期、导航结构、业务状态与原生能力接入的真相源。
- 对于**复杂布局、复杂表单、动态编辑区、需要高频调整视觉结构的页面**，优先考虑采用 **`WKWebView` 内嵌本地静态 React 资源** 的方案实现 UI。
- React 负责页面渲染、交互编排与局部乐观更新；Swift 负责状态真相源、校验、持久化、进程/系统能力与业务动作收口。
- 已 React 化的复杂页面应采用标准 Node / Vite 工程目录维护源码，并通过对应 `macos/scripts/build-*-webui.sh` 生成随 App bundle 分发的本地静态资源；不要手写长期维护的单体 `host.js` 作为 React 源码真相源。
- 原生 `dev` / release 构建入口需要先构建这些 React WebUI 资源，再进入 Swift 构建；App 运行时只加载生成后的本地静态资源，不在运行时依赖 Node。
- Ghostty、Workspace 主壳、Git / Commit / Diff 主链这类强原生交互区域，除非明确批准，不要为了“统一技术栈”而整体 React 化。

## 当前目录结构

- `dev`
  - 本机开发态入口；负责确保 `macos/Vendor` 可用（必要时复用同仓库其他 worktree 的 vendor）、接入 unified log 并运行 `swift run --package-path macos DevHavenApp`
- `release`
  - 本机 release 打包入口；固定委托 `bash macos/scripts/build-native-app.sh --release`，并透传其余参数
- `macos/Package.swift`
  - 原生子工程入口
- `macos/WebUI/WorkspaceRunConfiguration/`
  - 运行配置页的标准 React + Vite 源码工程；源码位于 `src/`，通过 `npm run build` 输出到 App 资源目录
- `macos/Sources/DevHavenApp/`
  - 原生 UI、窗口壳、GhosttyKit 宿主、设置页、终端工作区视图、Sparkle updater 运行时
- `macos/Sources/DevHavenApp/WorkspaceRunConfigurationResources/`
  - 运行配置 React 页面的构建产物；由 `macos/WebUI/WorkspaceRunConfiguration` 生成并随 App bundle 分发，Swift 侧 `WKWebView` 只加载这里的静态资源
- `macos/Sources/DevHavenApp/Update/`
  - Sparkle 相关的 bundle 元数据解析、appcast 手动检查、更新诊断与 updater controller
- `macos/Sources/DevHavenApp/AgentResources/`
  - 随 App bundle 分发的 Claude / Codex wrapper、Claude hook 与 signal emit 脚本
  - `shell/devhaven-agent-path.{zsh,bash}` 负责在用户 shell startup 可能重写 PATH 后重新把 Agent wrapper bin 目录归一化到 PATH 首位；不能只判断“路径是否存在”，因为用户 rc 可能把 Node / npm bin 再次顶到最前面
- `macos/Sources/DevHavenApp/DevHavenAppResourceLocator.swift`
  - App bundle 资源定位器；统一解析 `DevHavenNative_DevHavenApp.bundle`、`GhosttyResources` 与 `AgentResources`
- `macos/Sources/DevHavenApp/WorkspaceAgentStatusAccessory.swift`
  - 侧边栏 Agent 状态图标 / 文案映射；只负责展示语义，不负责状态聚合
- `macos/Sources/DevHavenApp/WorkspaceRunToolbarView.swift`
  - workspace 顶部右侧的轻量运行控制区；负责展示当前项目 `Project.runConfigurations` 运行配置菜单，以及 Run / Stop / Logs / 配置按钮，不直接持有进程或日志真相源
- `macos/Sources/DevHavenApp/WorkspaceRunConsolePanel.swift`
  - workspace 底部 Run Console；只负责按运行配置复用 tab、日志文本展示与“清空显示 / 打开日志 / 收起”入口，不直接启动或停止进程
- `macos/Sources/DevHavenApp/WorkspaceScriptConfigurationSheet.swift`
  - 当前项目的 typed 运行配置面板（`WorkspaceRunConfigurationSheet`）；负责编辑 `Project.runConfigurations`，首批支持 `customShell` 与 `remoteLogViewer`，不再依赖 Settings 里的通用脚本模板入口
- `macos/Sources/DevHavenApp/WorkspaceTerminalCommands.swift`
  - 工作区 terminal 的查找类菜单命令与 FocusedValue key；只负责把 App 菜单动作桥接到当前 focused pane，不持有 pane/runtime 真相源
- `macos/Sources/DevHavenApp/WorkspaceProjectCommands.swift`
  - 工作区“打开项目”菜单命令与 FocusedValue key；只负责把 App 菜单动作桥接到当前 workspace 壳层的 project picker 展示态，不持有 Core 业务状态真相源
- `macos/Sources/DevHavenApp/WorkspaceSplitTreeView.swift`
  - 工作区 pane 扁平布局渲染；根据 `WorkspacePaneTree` 的 leaf frame / split handle 结果平铺 pane 与 divider overlay，不再让 pane host 跟着递归 split 树迁移
- `macos/Sources/DevHavenApp/WorkspaceRootView.swift`
  - Workspace 根布局；负责项目导航与右侧 Workspace chrome 的 split、导航宽度持久化，以及“项目列表不属于工作区 chrome”这一层级关系
- `macos/Sources/DevHavenApp/WorkspaceProjectSidebarHostView.swift`
  - Workspace 外层项目导航宿主；负责已打开项目列表、project picker、worktree dialog 与删除 worktree 确认，不负责 terminal / Git 主内容
- `macos/Sources/DevHavenApp/WorkspaceChromeContainerView.swift`
  - Workspace 外围 chrome 容器；当前负责右侧工作区壳的 `左侧 tool window stripe | 主内容区` 布局与边框包裹，不再承载顶部栏或右侧辅助按钮区；默认不要再加外围额外 padding
- `macos/Sources/DevHavenApp/WorkspaceCommitSideToolWindowHostView.swift`
  - Commit 左侧独立工具窗宿主；负责 Commit 空态 / `WorkspaceCommitRootView` 路由、侧边焦点桥接，以及把 changes browser 的“单击同步已打开 preview / 双击打开或聚焦单实例 preview”统一桥接到 workspace runtime diff 标签页
- `macos/Sources/DevHavenApp/WorkspaceCommitRootView.swift`
  - Commit tool window 根容器；当前负责 changes browser + commit panel 双分区布局、初始化刷新，以及统一 preview 同步 / 打开闭包向下传递
- `macos/Sources/DevHavenApp/WorkspaceCommitChangesBrowserView.swift`
  - Commit tool window 左侧 changes browser；负责变更列表展示、inclusion toggle、选中项高亮，以及“单击同步已打开 preview / 双击打开或聚焦单实例 preview”；不要回退成一个文件一个 diff 标签页
- `macos/Sources/DevHavenApp/WorkspaceCommitDiffPreviewView.swift`
  - Commit diff preview 组件；当前文件保留以承载 diff preview 四态展示逻辑，但默认不再挂入 Commit tool window 根布局
- `macos/Sources/DevHavenApp/WorkspaceCommitPanelView.swift`
  - Commit tool window 底部 panel；负责提交信息编辑、基础 options（amend / sign-off / author）、主动作入口与 execution state 反馈
- `macos/Sources/DevHavenApp/WorkspaceHostView.swift`
  - Workspace 顶部标签宿主；负责统一渲染 terminal tab + runtime diff tab，并按 selected presented tab 在 terminal tree / diff viewer 之间切换；diff 内容区点击需回写 `.diffTab(...)` focused area
- `macos/Sources/DevHavenApp/WorkspaceDiffTabView.swift`
  - Workspace 独立 diff 标签页视图壳；负责 load state 路由、顶部 navigation bar 接线，以及把 patch / two-side / merge viewer 分发到独立子组件，不再自己拼 subtitle 或 pane header
- `macos/Sources/DevHavenApp/WorkspaceDiffNavigationBarView.swift`
  - Diff 顶部导航条；负责 Previous / Next Difference、差异/文件计数、viewer mode 切换与 refresh 入口，只消费 processor 输出的 navigator state
- `macos/Sources/DevHavenApp/WorkspaceDiffPaneHeaderView.swift`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zxcvbnmzsedr/devhaven](https://github.com/zxcvbnmzsedr/devhaven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
