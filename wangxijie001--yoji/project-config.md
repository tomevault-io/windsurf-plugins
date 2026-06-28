---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

本地 AI Agent 桌面应用。技术栈：Electron + React + TypeScript + Vite，基于 `electron-vite` 脚手架创建，包管理器 pnpm。

## 常用命令

```bash
pnpm install              # 安装依赖
pnpm dev                  # 启动开发环境（支持 HMR 热更新）
pnpm build                # 类型检查 + 构建
pnpm build:mac            # 构建并打包 macOS 应用
pnpm build:win            # 构建并打包 Windows 应用
pnpm format               # Prettier 格式化代码
pnpm lint                 # ESLint 检查
pnpm typecheck            # 运行所有 TypeScript 类型检查
```

## 三进程架构

Electron 应用由三个独立进程组成，源码分别对应 `src/` 下的三个目录：

### 1. 主进程 (`src/main/`)
Node.js 环境，负责窗口创建、系统交互、IPC 通信。**LangChain Agent、模型调用、文件读写、工具执行等核心逻辑全部在这里**。

```
src/main/
├── index.ts               # 入口：窗口创建 + 注册所有 IPC
├── config.ts              # 配置读写（getConfig/setConfig）
├── http.ts                # HTTP 请求工具
├── ipc/                   # IPC 处理器
│   ├── index.ts           # registerAll() 汇总注册
│   ├── agent.ts           # agent:chat / agent:chat:stream（流式对话）
│   ├── config.ts          # 配置 IPC（通用 config:getAll/set/delete）
│   ├── emotion.ts         # emotion:log（情绪日志查询）
│   ├── file.ts            # file:readAgentsMd / file:export / file:import
│   ├── http.ts            # HTTP IPC
│   ├── mcp.ts             # mcp:testConnection / mcp:updateMcpStoreVersion
│   ├── tts.ts             # tts:getEnabled / tts:setEnabled / tts:toggle
│   └── broadcast.ts       # 通用广播工具（主→渲染主动推送）
├── mcp/                   # MCP 客户端管理（预留）
└── agent/                 # AI Agent 核心
    ├── index.ts           # chat / chatStream（对话入口 + 流式处理）
    ├── create-agent.ts    # Agent 工厂（createAgent / getCheckpointer / companion 初始化，MCP 子 Agent 注入）
    ├── model.ts           # 模型工厂（DeepSeek / Qwen）
    ├── system-prompt.ts   # System Prompt 构建
    ├── children-agent/    # 子 Agent 定义
    │   └── mcp-execute-agent.ts  # MCP 工具执行子 Agent（SubAgent 定义 + getTools）
    ├── emotion/           # 激素情绪系统
    │   ├── schema.ts      # emotion_log 表操作（CRUD）
    │   ├── index.ts       # changeEmotion() 情绪变化引擎
    │   └── emotion_model.ts # LLM 情绪分析
    ├── tools/             # Agent 工具集（含 mcp-manage.ts MCP 管理工具）
    ├── skills/            # 内置 Skills
    ├── middleware/         # Agent 中间件（tool-error-handler: 工具调用容错）
    ├── children-agent/    # MCP 执行子 Agent
    └── utils/             # checkpoint 清理、chat-history、TTS 播报、token-logger、embedding 等
```

### 2. 预加载脚本 (`src/preload/`)
主进程和渲染进程之间的安全桥梁，通过 `contextBridge.exposeInMainWorld()` 将 API 暴露到渲染进程的 `window` 对象上。

```
src/preload/
├── index.ts               # contextBridge 注册入口
├── index.d.ts             # window.api 完整类型声明（所有 API 类型的真相源）
└── api/
    ├── index.ts           # 汇总所有 API 到 window.api
    ├── agent.ts           # chat / chatStream / historyQuery / onRebuilding
    ├── config.ts          # 配置读写（含 delete）
    ├── emotion.ts         # getLog / onUpdated
    ├── file.ts            # readAgentsMd / exportFile / importFile
    ├── http.ts            # HTTP 请求
    ├── mcp.ts             # testConnection / updateMcpStoreVersion
    ├── tts.ts             # getEnabled / setEnabled / toggle / onEnabledChanged
    └── listener.ts        # createListener<T>() 通用监听器工厂
```

### 3. 渲染进程 (`src/renderer/`)
普通 React 应用，运行在 Chromium 浏览器环境里。`@renderer` 别名映射到 `src/renderer/src/`。

```
src/renderer/
├── index.html               # HTML 入口（含 CSP 安全策略）
└── src/
    ├── main.tsx             # React 挂载点
    ├── App.tsx              # 根组件，HashRouter + Ant Design ConfigProvider
    ├── routes.tsx           # 路由配置（react-router-dom v7）
    ├── pages/
    │   ├── home/            # 首页（布局框架 + 菜单 + 情绪背景）
    │   ├── ai-chat/         # AI 对话页（无限滚动 + 流式输出）
    │   ├── modal-set/       # 模型配置页（DeepSeek / Qwen 切换）
    │   ├── param-show/      # 参数展示页
    │   ├── diary/           # 日记页
    │   ├── file-manage/     # 文件管理页（导出/导入记忆体）
    │   └── mcp-manage/      # MCP 服务器管理页（添加/测试/启用/卸载）
    ├── components/          # 通用组件（FormatChat、echarts 等）
    ├── api/                 # 渲染进程 API 封装层（处理 error toast）
    │   ├── agent.ts
    │   ├── config.ts
    │   ├── emotion.ts
    │   ├── file.ts
    │   ├── mcp.ts
    │   ├── tts.ts
    │   └── server.ts
    └── assets/              # CSS、SVG、iconfont 等静态资源
```

## IPC 通信模式

### 模式一：请求-响应（渲染 → 主）

```
渲染进程:  window.api.xxx(params)
预加载:    ipcRenderer.invoke('channel', params)
主进程:    ipcMain.handle('channel', async (event, params) => { ... })
```

用于渲染进程主动查询数据。示例：`agent:chat`、`emotion:log`、`file:readAgentsMd`。

返回统一格式：`{ ok: boolean; data?: T; error?: string }`。

### 模式二：主动推送（主 → 渲染）

```
主进程:    broadcast('channel', data)              // src/main/ipc/broadcast.ts
预加载:    createListener<T>('channel')             // src/preload/api/listener.ts
渲染进程:  const unsub = window.api.xxx.onUpdated(callback)
```

用于主进程在某个操作完成后主动通知渲染进程刷新 UI。示例：`insertEmotion` 后 `broadcast('emotion:updated', next)`。

`createListener` 返回的函数**自动返回取消订阅函数**，方便在 `useEffect` cleanup 中使用：

```ts
useEffect(() => {
  const unsub = window.api.emotion.onUpdated((emotion) => { ... })
  return unsub
}, [])
```

### 模式三：流式推送（主 → 渲染，多帧）

用于 AI 对话流式输出。主进程通过 `event.sender.send()` 分多次推送 chunk：

```
主进程:    event.sender.send('agent:stream:chunk', data)
预加载:    ipcRenderer.on('agent:stream:chunk', ...)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangxijie001/yoji](https://github.com/wangxijie001/yoji) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
