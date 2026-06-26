---
trigger: always_on
description: 灵犀 AI Agent 项目开发规范，适用于所有文件修改
---


# 灵犀 AI Agent — Cursor 开发规范

## 项目概述

灵犀 AI Agent 是一个本地优先的桌面 AI Agent 工作台。架构为 **Electron (桌面壳) + React (前端) + Go (后端)**，通过本地 AI 引擎与多模型接入层连接不同模型供应商。

## 关键规则

### 1. 子代理限制

**禁止在开发任务中开启子代理（subagent）。** 所有分析、编码、调试、打包任务必须在当前会话中直接完成。

### 2. 信令服务器代码推送规则

**凡对 `signaling-server/` 目录有代码变更，完成后必须 push 到独立仓库：**
```bash
# 克隆独立仓库
git clone git@github.com:OdysseyFather/lingxi-singaling-server.git /tmp/lingxi-singaling-server
# 复制变更文件
cp signaling-server/main.go signaling-server/go.mod signaling-server/go.sum /tmp/lingxi-singaling-server/
# 提交并推送
cd /tmp/lingxi-singaling-server && git add -A && git commit -m "描述变更" && git push origin main
```
Render 部署（wss://lingxi-singaling-server.onrender.com）会自动拉取最新代码重新部署。

### 3. 开发完成后的强制流程（每次改造后必须执行）

**每次需求开发完成后，必须按顺序执行以下全部步骤，不可跳过：**

1. **更新项目文档**
   - 更新 `.cursor/rules/lingxi-agent.mdc`（如有架构/规范变更）
   - 更新 `CLAUDE.md`（如有新增模块、技术栈变更、开发流程变更）
   - 更新 `README.md`（如有新增用户可见功能、快捷键、配置项等）

2. **打包编译**
   ```bash
   # 构建（需确保 Node.js >= 20.19 或 >= 22.12）
   export PATH="/tmp/node22/bin:$PATH"  # 如系统 node 版本不够
   cd <project-root> && ./build-desktop.sh
   ```
   构建产物在 `dist-electron/` 目录。

3. **安装验证**
   - macOS：打开 `dist-electron/mac-arm64/灵犀.app` 或安装 `.dmg`
   - 确认新功能可正常使用

**⚠️ 此流程为强制性规定，任何代码变更（无论大小）完成后都必须执行打包→安装，确保交付物始终可用。**

---

## 技术架构

### 前端 (`frontend-desktop/`)

| 技术 | 版本 | 用途 |
|------|------|------|
| React | 19 | UI 框架 |
| Vite | 8 | 构建工具（**需 Node.js ≥ 20.19 或 ≥ 22.12**） |
| Tailwind CSS | 3.4 | 样式系统 |
| Zustand | 5 | 全局状态管理 |
| Framer Motion | 12 | 动画库 |
| Lucide React | 1.14 | 图标库 |
| Recharts | 3 | 图表库 |
| react-markdown + remark-gfm | — | Markdown 渲染 |
| prism-react-renderer | 2 | 代码语法高亮 |
| @tanstack/react-virtual | 3 | 虚拟滚动 |

### 后端 (`backend-desktop/`)

| 技术 | 版本 | 用途 |
|------|------|------|
| Go | 1.24 | 语言 |
| Gin | 1.10 | HTTP 框架 |
| Gorilla WebSocket | 1.5 | WebSocket |
| ncruces/go-sqlite3 | 0.22 | SQLite 驱动（WASM 实现，无 CGO） |
| ledongthuc/pdf | — | PDF 文本提取 |
| nguyenthenguyen/docx | — | DOCX 文本提取 |

### 桌面壳 (`electron/`)

| 技术 | 版本 | 用途 |
|------|------|------|
| Electron | 36 | 桌面容器 |
| electron-builder | 25 | 打包工具 |

---

## 前端代码规范

### 组件体系

- **原子组件**在 `src/ui/primitives.jsx`：Button、Input、Textarea、Select、Badge、Card、Modal、ToastStack、Tooltip
- 所有新页面/组件必须使用 primitives 中的封装组件，不允许使用原生 HTML 元素 + 手写样式
- 使用 `cn()` 工具函数（`src/ui/cn.js`）合并 className，基于 `clsx` + `tailwind-merge`

### 样式规范

- **只使用 Tailwind CSS + CSS 变量**，不使用独立 CSS 文件（`App.css` 为遗留文件）
- 颜色使用 CSS 变量：`var(--bg)`, `var(--text)`, `var(--accent)`, `var(--line)` 等
- Tailwind 中引用 CSS 变量使用 `[color:var(--xxx)]` 语法
- 主题定义在 `src/index.css` 中，通过 `[data-theme="xxx"]` 选择器
- 当前主题：`light`, `dark`, `midnight`, `cyber`, `aurora`, `cosmos`

### 状态管理

- 全局状态使用 `useStore`（Zustand），定义在 `src/state/useStore.js`
- 页面级临时状态使用 `useState`
- 不引入 Redux / Context 等其他状态方案

### 动画

- 使用 `framer-motion` 的 `motion.div`、`AnimatePresence` 实现进场/退场动画
- 页面级切换在 `AppShell.jsx` 中通过 `AnimatePresence mode="wait"` 管理
- 列表增删使用 `AnimatePresence` + `motion.div` + `layout` 属性
- CSS 动画关键帧定义在 `src/index.css`（`shimmer`, `breathe`, `riseFade`, `pulseRing`, `rise`）

### 图标

- 统一使用 `lucide-react`，不使用 emoji 或其他图标库
- 图标尺寸：小按钮 `size={14}`，标准 `size={16-18}`，页面标题 `size={24-26}`

### 文件组织

```
src/
├── main.jsx              # 入口
├── index.css             # Tailwind 基础 + 主题 CSS 变量
├── api/client.js         # API 请求封装
├── state/
│   ├── useStore.js       # Zustand 全局状态（组合多切片）
│   └── slices/           # 状态切片（authSlice/uiSlice/sessionSlice/chatSlice/nexusSlice/codingSlice）
├── ui/                   # 通用 UI 组件
│   ├── AppShell.jsx      # 主布局壳（顶部导航：主导航6项 + 辅助导航5项）
│   ├── primitives.jsx    # 原子组件
│   ├── cn.js             # className 合并工具
│   ├── SidebarSessions.jsx # 会话列表（置顶/重命名/批量删除）
│   ├── ModelSwitcher.jsx
│   ├── RouterPill.jsx
│   └── ErrorBoundary.jsx
├── chat/                 # 对话相关
│   ├── ChatView.jsx      # 对话主视图
│   ├── Composer.jsx      # 输入框（含斜杠命令）
│   ├── MessageList.jsx   # 消息列表（含虚拟滚动）
│   ├── Bubble.jsx        # 消息气泡
│   ├── blocks.jsx        # 消息块渲染（文本/思考/工具）
│   ├── blockUtils.js     # 工具函数
│   ├── SearchModal.jsx   # 全文搜索弹窗
│   ├── AgentPicker.jsx   # 智能体选择器
│   ├── AgentStatePill.jsx
│   ├── ScreenBlock.jsx      # Screen Agent 屏幕截图 + 标注渲染
│   └── ScreenAgentPanel.jsx # Screen Agent 控制面板（截屏/规划/执行/确认）
├── settings/             # 设置页
│   ├── SettingsPage.jsx
│   ├── ProfilesPage.jsx  # 接入点管理
│   ├── AppearancePage.jsx # 主题设置
│   ├── MemoryPage.jsx    # 长期记忆管理
│   ├── NexusSettingsPage.jsx # 网络与协作设置
│   └── UsagePage.jsx     # 用量统计 + 预算预警
├── ModeSelector.jsx      # 启动模式选择页（灵犀主模式 vs Coding Agent）
├── code/                 # 编程视图（Coding Agent，独立模式）
│   ├── CodingShell.jsx   # 独立模式布局壳（tab栏+图标栏+侧边栏+主区域+Changes面板+状态栏）
│   ├── CodingChatView.jsx # 对话主视图（cc-haha 风格渲染）
│   ├── CodingComposer.jsx # 输入栏（文件chip+文件浏览器+模型选择器+Run/Stop）
│   ├── CodingToolCard.jsx # 工具调用卡片（颜色编码 + diff 渲染）
│   ├── CodingIconBar.jsx # 左侧极窄图标栏（40px）
│   ├── CodingSidebar.jsx # 左侧会话侧边栏（搜索+日期分组）
│   ├── CodingTabBar.jsx  # 顶部多tab会话栏
│   ├── SessionHeader.jsx # 会话标题+状态信息
│   ├── AskQuestionBlock.jsx # Agent提问交互块
│   ├── PermissionBlock.jsx # 权限确认块
│   ├── TaskTodoList.jsx  # Task Todo List面板
│   ├── AgentTeamPanel.jsx # Agent Teams协作面板
│   ├── WorkspaceChanges.jsx # 右侧文件变更面板
│   ├── DiffViewer.jsx    # Diff渲染组件
│   ├── FileChip.jsx      # 文件附件chip组件
│   ├── BottomStatusBar.jsx # 底部状态栏

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OdysseyFather/lingxi](https://github.com/OdysseyFather/lingxi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
