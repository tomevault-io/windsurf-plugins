---
trigger: always_on
description: Electron + Vue 3 内容创作桌面平台，通过 MCP Server（子进程 HTTP 模式）接入 27 个图像处理/文件操作工具。
---

# nowCreate-electron

Electron + Vue 3 内容创作桌面平台，通过 MCP Server（子进程 HTTP 模式）接入 27 个图像处理/文件操作工具。

---

## ⚠️ 关键陷阱（新会话必读）

### 1. 系统环境变量 ELECTRON_RUN_AS_NODE=1

该机器上将此变量设为了**系统级环境变量**，值 `=1`。Electron 判断该变量**是否存在**（不判断值），存在即强制以 Node.js 模式运行，后果：

- `require('electron')` / `import ... from 'electron'` 返回 `node_modules/electron/index.js` 导出的**字符串路径**（指向 electron.exe），而非 Electron API 对象
- `app` `BrowserWindow` `ipcMain` 等全部为 `undefined`
- Electron 窗口无法创建，所有 Electron API 调用崩溃

**正确启动方式**：

```bash
# ✅ 通过项目封装脚本启动（推荐）
yarn dev

# ✅ 手动 unset 后直接启动
unset ELECTRON_RUN_AS_NODE && npx electron-vite dev

# ❌ 不要直接运行 electron.exe（会继承系统环境变量）
npx electron .                            # 报错：electron.app.whenReady is not a function
./node_modules/.bin/electron .            # 同上
```

`scripts/start-dev.js` 通过 `delete env.ELECTRON_RUN_AS_NODE` 清除该变量后 spawn electron-vite，**`yarn dev` 已自动处理此问题**。若需直接跑 electron 命令调试，务必先 `unset ELECTRON_RUN_AS_NODE`。

**排查标志**：报错信息中出现以下任一即为此问题 ——
- `TypeError: Cannot read properties of undefined (reading 'exports')`（ESM 模式）
- `TypeError: Cannot read properties of undefined (reading 'whenReady')`（CJS 模式）
- `require('electron')` 返回字符串而非对象

### 2. 不要手动改 package.json 的 "type" 字段

项目根 `package.json` 设 `"type": "module"`。删除该字段会导致 electron-vite 将主进程打包为 CJS（`require` 语法），与 Electron 内嵌的模块加载器产生新的冲突。**该字段为构建系统所用，勿动。**

### 3. MCP Server 路径硬编码

`src/main/mcp-launcher.js` 中 `MCP_SERVER_DIR` 硬编码为 `D:\\leh_project\\sundries\\mcp-server`。生产打包前需改为动态路径（如 `path.join(app.getAppPath(), 'mcp-server')` 并将 MCP Server 目录复制到打包产物中）。

### 4. Node 版本差异

| 环境 | Node 版本 |
|------|-----------|
| 系统 | v18.20.8 |
| Electron 内嵌 | v20.18.0 |
| MCP Server 要求 | >= 18.0.0 |

注意 `import.meta.dirname` 在 Node 18 不可用（21.2+ 才支持），主进程/脚本中使用 `dirname(fileURLToPath(import.meta.url))` 替代。

---

## 目录结构

```
nowCreate-electron/
├── package.json                     # 依赖 + 脚本
├── electron.vite.config.mjs         # electron-vite 构建配置（main/preload/renderer）
├── mcp.json                         # 外部 AI 客户端 MCP 配置模板
│
├── scripts/
│   └── start-dev.js                 # 开发启动封装（清理 ELECTRON_RUN_AS_NODE）
│
├── src/main/                        # Electron 主进程
│   ├── index.js                     # 入口 → app.whenReady 流程
│   ├── mcp-client.js                # MCP HTTP 客户端（session 管理）
│   ├── mcp-launcher.js              # MCP Server 子进程生命周期
│   └── mcp-ipc.js                  # IPC 桥接（ipcMain.handle → HTTP 转发）
│
├── src/preload/
│   └── index.js                     # contextBridge → window.mcp
│
├── src/renderer/                    # Vue 3 渲染进程
│   ├── index.html                   # Vite 入口 HTML
│   └── src/
│       ├── main.js                  # Vue 根实例（Element Plus + Pinia + Router）
│       ├── App.vue                  # el-config-provider 包裹
│       ├── router/index.js          # Vue Router（hash 模式）
│       ├── stores/index.js          # Pinia（useProjectStore / useUiStore）
│       ├── composables/
│       │   └── useMcpTools.js       # window.mcp 调用封装
│       ├── components/
│       │   ├── layout/index.vue     # 主布局（顶栏 + 侧面板 + 画布 + AI 聊天）
│       │   ├── canvas/index.vue     # 画布区域（占位）
│       │   └── chat/index.vue       # AI 聊天面板
│       └── views/
│           ├── home/index.vue       # 首页
│           └── system/notFind.vue   # 404
│
└── resources/                       # 打包资源（icon 等）
```

---

## 启动命令

```bash
yarn dev            # 开发模式（electron-vite dev + HMR）
yarn build          # 生产构建
yarn preview        # 预览构建产物
```

首次安装依赖（如缺少 node_modules 或 yarn.lock）：

```bash
ELECTRON_MIRROR=https://npmmirror.com/mirrors/electron/ yarn install
```

---

## 架构

```
┌─────────────────────────────────────────────────────────────────┐
│  Electron App                                                    │
│                                                                  │
│  ┌──────────────────┐     IPC      ┌──────────────────────────┐  │
│  │  渲染进程          │◄──────────►│  主进程                    │  │
│  │  Vue 3 + Element  │  window.mcp │  ipcMain.handle('mcp:*')  │  │
│  │  ChatPanel.vue    │             │  mcp-ipc.js               │  │
│  │  Canvas.vue       │             │         │                  │  │
│  │  LayerPanel.vue   │             │    HTTP POST /mcp          │  │
│  └──────────────────┘             │         ▼                  │  │
│                                    │  ┌──────────────────┐     │  │
│                                    │  │ MCP Server 子进程  │     │  │
│                                    │  │ localhost:9100    │     │  │
│                                    │  │ (HTTP 模式)       │     │  │
│                                    │  │        │          │     │  │
│                                    │  │   execFile        │     │  │
│                                    │  │        ▼          │     │  │
│                                    │  │  nxImage.exe      │     │  │
│                                    │  │  (27 tools)       │     │  │
│                                    │  └──────────────────┘     │  │
│                                    └──────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  外部 AI 接入（可选）                                       │    │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nx2026005/zhijian_electron_02](https://github.com/nx2026005/zhijian_electron_02) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
