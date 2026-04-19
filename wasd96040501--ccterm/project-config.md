---
trigger: always_on
description: 纯 SwiftUI 架构，最低部署目标 macOS 14（Sonoma）。
---

# CCTerm 代码规范

## 架构

纯 SwiftUI 架构，最低部署目标 macOS 14（Sonoma）。

入口：`@main CCTermApp` → `Window` scene → `RootView`（`NavigationSplitView`）→ Sidebar + Content。

- **Model**: 纯数据结构，`struct` 优先，`Codable` 协议
- **View**: SwiftUI View struct，声明式 UI
- **ViewModel**: `@Observable` 类，持有页面状态和业务逻辑
- **AppState**: `@Observable` 全局状态容器，持有 Services、ChatRouter 和 ViewModels，通过 `.environment()` 注入

不使用 XIB / Storyboard / NSHostingController。需要 AppKit 能力时通过 `NSViewRepresentable` 桥接。

### SwiftUI 规范

**状态管理：**
- MVVM：`@Observable` ViewModel 持有页面状态和业务逻辑，View 只负责声明式渲染和用户交互回调
- `@State`：View 私有 UI 状态；`@Binding`：父传子可写引用
- Service 通过 init 注入，View 不自行创建 Service
- 通用 SwiftUI 组件放在 `Components/` 目录

**View 编写：**
- body 超 40 行应拆分——有独立状态的提取为子 View struct，纯布局的提取为计算属性或 `@ViewBuilder` 方法
- 有独立状态的子 View 提取为独立文件；无状态的辅助类型（Context、小 enum）可留在主文件中。通用 ViewModifier / 通用组件放 `Components/`
- 禁止在 body 中做昂贵计算；长列表用 `List` / `LazyVStack`；`ForEach` 的 `id` 必须稳定
- 数据加载用 `.task { }`，依赖变化用 `.task(id:)` / `.onChange(of:)`，禁止在 body 构建路径中触发副作用

## Swift↔React Bridge

聊天渲染使用 WKWebView 内嵌 React（源码在 `web/`）。双端通过 `WebViewBridge` 通信。

| 层 | 职责 |
|----|------|
| Swift (SessionHandle + MessageFilter) | 数据处理：AgentSDK 消息解析、会话状态管理、生成渲染就绪的 ChatMessage |
| Bridge (WebViewBridge + bridge.ts) | 传输：callAsyncJavaScript + JSON 双向传递，不含业务逻辑 |
| React | 纯渲染：收到 string 就渲染，不做数据处理 |

**通信方向：**
- Swift → React：`callAsyncJavaScript` 调用 `window.__bridge(type, json)`，JSON string 作为参数
- React → Swift：`window.webkit.messageHandlers.bridge.postMessage(event)`，JSON 对象

**类型定义：**
- Swift + React：消息类型（Message2 enum, Message2User, Message2Assistant）由 schema 自动生成至 `AgentSDK/Sources/AgentSDK/Generated/` 和 `web/src/generated/types.generated.ts`
- Bridge 类型：Swift `Services/WebViewBridge.swift` + React `types/bridge.ts`
- 修改共享类型时**必须同步更新两侧**（自动生成的类型修改 schema 即可）

**规范：**
- 消息用 enum（Swift）/ discriminated union（TS）区分 user/assistant，不用 role 字段
- 禁止手工拼接 JS 字符串或转义，必须用 callAsyncJavaScript 传递 JSON 参数
- 禁止在 React 侧暴露 store 函数到 window，外部入口统一通过 bridge.ts
- React 不做数据处理，Swift 预处理文本，React 收到 string 直接渲染
- 新增 Swift→React 交互：Bridge 加 typed 方法 + React 侧 onNativeEvent 加分支

## Chat 架构

Chat 模块采用 ChatRouter + per-session ChatSessionViewModel 架构。

### 核心实体

| 实体 | 类型 | 数量 | 职责 |
|------|------|------|------|
| ChatRouter | @Observable | 1 | session 生命周期路由和协调：切换 session、消息提交路由（新建/恢复/直发）、per-session ViewModel 缓存管理。不持有 UI 状态 |
| ChatSessionViewModel | @Observable | N（per-session） | 单个 session 的完整 UI 状态：computed 直读 SessionHandle 运行时状态，stored 管理本地 UI 状态，action 直调 handle 方法。InputBar 的唯一数据源 |
| ChatInputBarActions | struct | 1 个闭包 | 仅 `onSend`（需要 ChatRouter 路由），其余操作 InputBar 直调 `sessionVM.method()` |

### 持有关系

```
AppState
├── sessionService: SessionService
│   └── handles: [String: SessionHandle] ×N
│       ├── agentSession: AgentSDK.Session?
│       └── bridge: WebViewBridge?（创建时绑定，归档/删除时解绑，stop 不动）
├── sidebarViewModel: SidebarViewModel
│   └── 依赖: sessionService（观察 allHandles，自驱动 rebuild）
│   └── ❌ 不认识 ChatRouter，不持有选中状态
├── chatRouter: ChatRouter
│   ├── currentSession: ChatSessionViewModel（当前活跃实例）
│   ├── sessions: [String: ChatSessionViewModel]（per-session 缓存）
│   ├── chatContentView: ChatContentView
│   │   └── bridge: WebViewBridge
│   └── 依赖: sessionService
└── todoSessionCoordinator: TodoSessionCoordinator
```

### 数据流

- **状态渲染**：SessionHandle.status 变化 → ChatSessionViewModel.barState（computed 直读）→ SwiftUI 自动追踪 → InputBar 重渲染。无手动 observation，无逐字段拷贝
- **消息推送**：所有 handle（包括后台）实时推送消息到 bridge → React 按 conversationId 分存
- **Session 切换**：SidebarView onSelect → chatRouter.switchSession(id) → 整体替换 currentSession 实例 → bridge.switchConversation(id)。选中状态 source of truth 是 `chatRouter.currentSession.sessionId`
- **消息提交**：InputBar → actions.onSend → ChatRouter.submitMessage 路由：handle == nil → startNewSession / handle.status == .inactive → resumeSession / 其他 → handle.send

### UI 层胶水

RootView 是唯一同时知道 SidebarViewModel 和 ChatRouter 的地方。SidebarView 通过 `selection` 参数只读获取当前选中 ID，通过 `onSelect` / `onArchive` 闭包回调 ChatRouter。ChatView 只依赖 ChatRouter。

### 规范

- 禁止在 View/ViewModel 中直接修改 session 的 processing/interrupting 状态，必须通过 SessionHandle 方法触发
- UI 层只读取和映射状态，不维护独立的 session 状态
- InputBar 的操作（interrupt、selectModel、queueMessage 等）直调 `sessionVM.method()`，仅 `onSend` 经过 ChatRouter 路由
- 需要 ChatRouter 协调的操作通过 `onRouterAction` 回调传递 `ChatRouterAction` enum，不用瞬态属性 + `.onChange` 中转
- 新增 session 运行时状态：在 SessionHandle 加字段 → ChatSessionViewModel 加 computed 直读（SwiftUI 自动追踪）

## Session 运行时状态

`SessionHandle`（`@Observable`，定义在 `Services/Session/SessionHandle.swift`）是单个会话的运行时句柄，持有可观察状态和交互命令。

`SessionService`（定义在 `Services/Session/SessionService.swift`）管理 SessionHandle 的生命周期（创建/缓存/start/stop/remove），是 handles 字典的 owner。

### Observe vs Event

SessionHandle 的对外接口严格区分两种机制：

| 机制 | 用途 | 消费方式 | 示例 |
|------|------|----------|------|
| **Observe**（`@Observable` 属性） | 连续状态 → 驱动 UI 渲染 | SwiftUI 自动追踪 / computed 直读 | status、branch、contextUsedTokens、pendingPermissions |
| **Event**（`SessionEvent` + `AsyncStream`） | 离散事件 → 触发副作用 | ViewModel `Task { for await event in handle.eventStream() }` | statusChanged、permissionsChanged、processExited |

**判断标准：** 如果消费者需要「当前值是什么」→ Observe；如果消费者需要「刚刚发生了什么」→ Event。

**规范：**
- 禁止在 View/ViewModel 中缓存 SessionHandle 属性副本作为状态来源，通过 computed 直读 handle 属性
- 本地操作（send/interrupt/setPermissionMode）只写 stdin 请求 CLI 变更，不直接改 observable 值

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wasd96040501) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
