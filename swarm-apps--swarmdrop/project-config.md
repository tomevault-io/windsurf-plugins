---
trigger: always_on
description: > 本文件为 AI Coding Agent 提供项目背景、架构说明和开发规范。
---

# SwarmDrop 项目指南

> 本文件为 AI Coding Agent 提供项目背景、架构说明和开发规范。

## 项目概述

**SwarmDrop** 是一款去中心化、跨网络、端到端加密的文件传输工具，定位为"跨网络版的 LocalSend"。无需账号、无需服务器，支持局域网和跨网络点对点文件传输。

- **当前阶段**: Phase 2 (设备配对系统) —— 网络层已完成，配对系统进行中
- **应用标识**: `com.yexiyue.swarmdrop`
- **主语言**: 简体中文（所有注释和文档）

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端框架 | React 19 + TypeScript 5.8 + Vite 7 |
| 样式 | Tailwind CSS 4 |
| 路由 | TanStack Router（文件系统路由，自动代码分割） |
| 状态管理 | Zustand 5（4 个 Store：auth、network、preferences、secret） |
| UI 组件 | shadcn/ui（new-york 风格）+ Radix UI + Lucide 图标 |
| 国际化 | Lingui 5（8 语言：zh, zh-TW, en, ja, ko, es, fr, de） |
| 后端 | Rust 2021 + Tauri 2 |
| P2P 网络 | libp2p 0.56（通过 `swarm-p2p-core` 子模块） |
| 安全 | Stronghold（加密密钥库）+ Biometry（FaceID/TouchID/Windows Hello） |

## 项目结构

```
swarmdrop/
├── src/                          # 前端源码
│   ├── commands/                 # Tauri IPC 调用封装
│   ├── components/               # React 组件
│   │   ├── ui/                   # shadcn/ui 组件
│   │   ├── layout/               # 布局组件
│   │   ├── devices/              # 设备相关组件
│   │   ├── network/              # 网络状态组件
│   │   └── pairing/              # 配对流程组件
│   ├── hooks/                    # 自定义 React Hooks
│   ├── lib/                      # 工具函数和库封装
│   ├── locales/                  # 国际化翻译文件 (.po)
│   ├── routes/                   # TanStack Router 路由页面
│   │   ├── __root.tsx            # 根布局
│   │   ├── _auth.tsx             # 未认证布局（Aurora 背景）
│   │   ├── _auth/                # 认证流程页面
│   │   ├── _app.tsx              # 已认证布局（侧边栏/底部导航）
│   │   └── _app/                 # 主应用页面
│   ├── stores/                   # Zustand 状态管理
│   └── main.tsx                  # 应用入口
├── src-tauri/                    # Tauri Rust 后端
│   ├── src/
│   │   ├── lib.rs                # 主入口，插件注册，命令处理器
│   │   ├── commands/             # Tauri 命令处理器
│   │   ├── device/               # 设备管理
│   │   ├── network/              # 网络管理（P2P 节点）
│   │   ├── pairing/              # 配对系统
│   │   ├── protocol.rs           # P2P 协议定义
│   │   └── error.rs              # 错误类型定义
│   ├── capabilities/             # Tauri 权限配置
│   └── Cargo.toml                # Rust 依赖
├── libs/                         # Git 子模块：P2P 核心库
│   └── core/                     # swarm-p2p-core crate
├── docs/                         # Astro + Starlight 文档站点
├── dev-notes/                    # 开发文档
└── skills/                       # Claude Code 技能文件
```

## 构建和开发命令

```bash
# 包管理器：pnpm（不要使用 npm 或 yarn）

# 完整应用开发（Vite 前端 + Tauri Rust 后端）
pnpm tauri dev

# 仅前端开发（Vite dev server，端口 1420）
pnpm dev

# 生产构建
pnpm build              # 前端构建（tsc + vite build）
pnpm tauri build        # 完整应用构建

# 国际化
pnpm i18n:extract       # 提取翻译字符串到 .po 文件

# Rust 命令（需在 src-tauri/ 目录下执行）
cargo build
cargo test
cargo clippy
cargo fmt
```

## 架构详情

### 前端 ↔ 后端通信

前端通过 Tauri IPC 调用 Rust 后端。TypeScript 封装位于 `src/commands/`：

```typescript
// src/commands/network.ts
import { invoke } from "@tauri-apps/api/core";

export async function startNode(keypair: Keypair, devices: PairedDeviceInfo[]) {
  return await invoke<void>("start", { keypair, pairedDevices: devices });
}
```

Rust 命令处理器位于 `src-tauri/src/commands/`，在 `lib.rs` 中注册：

```rust
.invoke_handler(tauri::generate_handler![
    commands::start,
    commands::shutdown,
    commands::generate_keypair,
    // ...
])
```

### 路由系统

使用 TanStack Router 文件系统路由：

| 文件模式 | 含义 |
|----------|------|
| `__root.tsx` | 根布局 |
| `_layout.tsx` | 无路径布局（路径中不包含 `_layout`） |
| `page.lazy.tsx` | 懒加载路由（代码分割） |
| `index.tsx` | 目录索引路由 |

**当前路由结构：**
- `__root.tsx` — 根布局
- `_auth.tsx` — 未认证布局（Aurora 背景），守卫重定向到 `/devices`
- `_auth/welcome.lazy.tsx` — 欢迎页
- `_auth/setup-password.lazy.tsx` — 设置密码
- `_auth/unlock.lazy.tsx` — 解锁页
- `_auth/enable-biometric.lazy.tsx` — 启用生物识别
- `_app.tsx` — 已认证布局（侧边栏/底部导航）
- `_app/devices.lazy.tsx` — 设备列表
- `_app/settings.lazy.tsx` — 设置页
- `index.tsx` — 重定向到 `/devices`

### 状态管理

4 个 Zustand Store，不同持久化策略：

| Store | 用途 | 持久化 |
|-------|------|--------|
| `auth-store` | 认证流程状态 | `localStorage`（仅 `isSetupComplete` + `biometricEnabled`） |
| `preferences-store` | 主题、语言、设备名称 | `tauri-plugin-store` |
| `secret-store` | Ed25519 密钥对 | Stronghold 加密密钥库 |
| `network-store` | P2P 节点状态、对等节点列表 | 仅运行时（不持久化） |

### 响应式设计

3 个断点（`use-breakpoint` hook）：
- **mobile** (<768px): 底部导航
- **tablet** (768–1023px): 图标-only 侧边栏
- **desktop** (≥1024px): 展开侧边栏

### 国际化

使用 Lingui 框架，Babel macro 提取翻译：

```tsx
import { msg, Trans } from "@lingui/macro";

// JSX 中使用
<Trans>欢迎使用 SwarmDrop</Trans>

// 代码中使用
const message = i18n.t(msg`设备已连接`);
```

源语言为简体中文 (`zh`)。提取命令：`pnpm i18n:extract`

翻译文件位置：`src/locales/{locale}/messages.po`

### P2P 网络架构

**启动流程：**
1. `commands::start()` 创建 `NodeConfig`（启用 mDNS、Relay、DCUtR、autonat、引导节点）
2. 调用 `swarm_p2p_core::start::<AppRequest, AppResponse>()` → 返回 `(NetClient, Receiver<NodeEvent>)`
3. 生成 tokio 任务执行 DHT bootstrap
4. 创建 `NetManager`，存入 Tauri state
5. 启动事件循环，通过 Tauri Channel 转发事件到前端

**引导节点**: `47.115.172.218:4001`（TCP + QUIC）

**分享码系统**: 6 位数字，DHT key = SHA256(code)，记录包含 OS 信息 + 时间戳，默认 TTL 300 秒

> 移动端（iOS / Android）已迁移到独立的 **SwarmDrop-RN** 项目（React Native + Expo + uniffi），
> 共用 `crates/core` 与 `libs/core`。本仓库自此只保留桌面端 (Windows / macOS / Linux)。

## 开发规范

### 代码风格

- **注释**: 使用简体中文
- **命名**:
  - Rust：模块使用中文文档注释，代码使用英文命名
  - TypeScript：同样规范
- **路径别名**: `@/` 映射到 `./src/`（TypeScript 和 Vite 中一致）

### 错误处理


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swarm-apps/SwarmDrop](https://github.com/swarm-apps/SwarmDrop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
