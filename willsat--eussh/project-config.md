---
trigger: always_on
description: **eussh** 是一个跨平台 SSH 客户端桌面应用，基于 **Tauri v2** + **Vue 3** 构建，版本 `1.3.7`。
---

# AGENTS.md

## 项目概览

**eussh** 是一个跨平台 SSH 客户端桌面应用，基于 **Tauri v2** + **Vue 3** 构建，版本 `1.3.7`。

核心功能：多服务器 SSH 连接（Tab 页管理）、终端仿真（xterm.js）、SFTP 文件管理、服务器资源监控面板（CPU/内存/磁盘/网络）、批量命令执行、主机密钥 TOFU 验证、配置加密存储、中英双语支持。

---

## 技术栈

| 层级 | 技术 | 关键依赖 |
|------|------|----------|
| 桌面框架 | Tauri v2 | Rust 系统 WebView |
| 前端 | Vue 3 + Pinia 3 + Vite 8 | xterm.js 6, ECharts 6, Tailwind CSS 4 |
| SSH 后端 | Rust (edition 2021) | `russh` 0.61 (pure Rust SSH), `tokio` |
| 加密存储 | AES-256-GCM + PBKDF2 | `aes-gcm`, `ring` (machine-id 派生密钥) |

---

## 目录结构与职责

```
eussh/
├── src/                          # Vue 前端源码
│   ├── main.js                   # 入口：创建 Vue App + Pinia
│   ├── App.vue                   # 根组件，渲染 AppShell
│   ├── assets/css/main.css       # 全局样式 + Tailwind CSS
│   ├── components/
│   │   ├── common/               # 通用组件：Modal, Toast, RoseSpinner
│   │   ├── connection/           # 连接对话框、主机密钥验证对话框
│   │   ├── filemanager/          # 文件管理器（含右键菜单、面包屑、图标/列表视图）
│   │   ├── layout/               # 布局组件：AppShell, ActivityBar, Sidebar, MainTabBar, StatusBar
│   │   │   └── sidebar/          # 侧边栏视图：ServersView, BatchView, SettingsView
│   │   ├── server/               # 服务器概览面板（CPU/内存/磁盘/地图/IP 列表）
│   │   └── terminal/             # 终端容器（xterm.js 实例管理）
│   ├── composables/              # Vue Composables（复用逻辑）
│   │   ├── useXterm.js           # xterm.js 生命周期管理
│   │   ├── useI18n.js            # 国际化（中/英，支持 {{param}} 模板）
│   │   ├── useTheme.js           # 暗色/亮色主题切换
│   │   ├── useToast.js           # Toast 通知队列
│   │   ├── useLogger.js          # 应用内调试日志（最多 200 条）
│   │   └── useServerData.js      # 服务器动态数据拉取（监控指标 + 静态信息）
│   ├── stores/                   # Pinia 状态管理
│   │   ├── useConnectionStore.js # 连接配置 CRUD
│   │   ├── useServerStore.js     # 运行时服务器状态（Tab、重连、Ping/流量）
│   │   ├── useSettingsStore.js   # 用户偏好设置
│   │   └── useFileManagerStore.js# 文件管理器状态（路径、选中、剪贴板）
│   ├── utils/
│   │   ├── ipc.js                # Tauri IPC 封装（invoke 统一错误处理）
│   │   └── theme.js              # 终端配色方案预设（6 套主题）
│   └── locales/                  # 国际化语言文件
│       ├── en.js
│       └── zh-CN.js
│
├── src-tauri/                    # Rust 后端源码
│   ├── tauri.conf.json           # Tauri 配置（窗口、CSP、打包）
│   ├── Cargo.toml                # Rust 依赖声明
│   ├── capabilities/default.json # Tauri 权限清单
│   └── src/
│       ├── main.rs               # 入口：创建 AppState，注册全部 23 个 Tauri 命令
│       ├── state.rs              # AppState（持有 ConfigStore + SshManager）
│       ├── commands/             # Tauri IPC 命令处理器
│       │   ├── mod.rs
│       │   ├── config.rs         # get_config, save_config, save/delete_connection
│       │   ├── connection.rs     # connect, disconnect, terminal_write/resize, exec, ping, traffic, clipboard, batch_exec, confirm_host_key
│       │   ├── file.rs           # file_list, mkdir, remove, rename, copy, exists, read, write, download_dir, upload_path, chmod
│       │   └── open.rs           # open_url（严格 URL 校验，仅允许 HTTP/HTTPS）
│       ├── ssh/                  # SSH 协议实现
│       │   ├── mod.rs
│       │   ├── manager.rs        # SshManager：会话生命周期、exec 队列、Ping 限速
│       │   ├── session.rs        # SshSession：DNS → TCP 连接 → SSH 握手 → 认证 → PTY → Shell → 读写循环
│       │   └── host_key.rs       # HostKeyVerificationManager：known_hosts 管理、TOFU 验证
│       ├── storage/              # 持久化存储
│       │   ├── mod.rs
│       │   ├── config_store.rs   # ConfigStore：原子读写（tmp 写入 + rename）
│       │   └── encrypt.rs        # AES-256-GCM 加密 + PBKDF2 密钥派生
│       └── models/               # 数据结构
│           ├── mod.rs
│           ├── connection.rs     # ConnectionProfile, AuthMethod（密码 / 私钥）
│           └── config.rs         # AppConfig, AppSettings（字体、配色、监控间隔等）
│
├── public/world.json             # GeoJSON 世界地图数据（Natural Earth 110m）
├── index.html                    # HTML 入口（含主题闪烁防护脚本）
├── vite.config.js                # Vite 配置（端口 5173，`@` 别名指向 src）
├── package.json                  # NPM 配置
└── .github/workflows/            # CI/CD
    ├── build.yml                  # PR/Push 构建（macOS ARM64, Ubuntu x86_64, Windows MSVC）
    └── release.yml               # Tag 触发发布（多平台构建 + GitHub Release）
```

---

## 架构与数据流

### 前后端通信

```
Vue 前端                          Rust 后端
─────────                        ─────────
invoke('command', args)  ──→    Tauri IPC 命令处理器
                              ↓
event.listen('event')  ←──    app_handle.emit('event', data)
```

- **Request/Response**：前端通过 `invoke()` 调用后端命令（`src-tauri/src/commands/` 下的函数），返回 Promise
- **Event Push**：后端主动推送事件到前端（`connection-status`, `terminal-data`, `host-key-verify`, `debug-event`, `ping` 等）

### 前端 IPC 封装（`src/utils/ipc.js`）

```js
import { invoke } from '@/utils/ipc'
const result = await invoke('get_config')
```

所有 Tauri invoke 调用必须通过此封装，统一错误处理和日志。

### 状态管理（Pinia Stores）

| Store | 职责 | 文件 |
|-------|------|------|
| `useConnectionStore` | 连接配置的 CRUD，通过 IPC 读写加密配置 | `src/stores/useConnectionStore.js:7` |
| `useServerStore` | 运行时服务器状态：连接/断开、Tab 管理、重连逻辑、Ping/流量定时器 | `src/stores/useServerStore.js:18` |
| `useSettingsStore` | 用户偏好（主题、字体、终端配色等），从加密配置加载 | `src/stores/useSettingsStore.js` |
| `useFileManagerStore` | 文件管理器状态：当前路径、文件列表、选中项、剪贴板、导航历史 | `src/stores/useFileManagerStore.js` |

### SSH 会话生命周期

```
connect() → DNS 解析 → TCP 连接 (10s 超时)
→ SSH 握手 (15s 超时) → 主机密钥验证 (TOFU)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WillSat/eussh](https://github.com/WillSat/eussh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
