---
trigger: always_on
description: 本文件为 Claude Code (claude.ai/code) 在此代码仓库中工作时提供指导。
---

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此代码仓库中工作时提供指导。

## 项目概述

HisCRM-IM 是一个基于 Master-Worker 架构的多平台社交媒体监控系统。支持实时监控抖音等平台的评论和私信，并通过桌面端（Electron）和移动端（React Native）客户端发送智能通知。

**核心创新**：每个被监控的账户都运行在独立的浏览器进程中，具有独特的浏览器指纹，实现 100% 隔离以避免平台检测。

## 系统架构

系统由三个主要层级组成：

1. **Master 主控服务器** (`packages/master`, 端口 3000)
   - 中央协调器，管理 Worker 生命周期、任务调度和客户端通信
   - SQLite 数据库持久化 (`packages/master/data/master.db`)
   - Socket.IO 服务器，三个命名空间：`/admin`、`/worker`、`/client`

2. **Worker 工作进程** (`packages/worker`, 端口 4000+)
   - 使用 Playwright 进行浏览器自动化（注意：不是 Puppeteer，尽管 package.json 中有相关依赖）
   - 平台特定爬虫位于 `src/platforms/`（抖音、小红书等）
   - 多浏览器隔离：每个账户 = 独立的浏览器进程
   - React Fiber 数据提取技术用于虚拟列表组件

3. **客户端层**
   - Admin Web UI (`packages/admin-web`, 端口 3001) - React 18 + Ant Design
   - CRM PC IM (`packages/crm-pc-im`) - Electron 桌面客户端 + Vite
   - CRM IM Server (`packages/crm-im-server`) - 传统 WebSocket 服务器，用于 PC/移动端

4. **共享代码** (`packages/shared`)
   - 协议定义位于 `protocol/messages.js` 和 `protocol/events.js`
   - 数据模型和工具函数

## 开发命令

### 初始化设置
```bash
# 安装所有依赖（根目录 + 所有 packages）
npm run install:all

# 或手动安装
npm install
npm run install:packages
```

### 运行服务

```bash
# 启动 Master 服务器（端口 3000）
npm run start:master
# 或：cd packages/master && npm start

# 启动 Worker 进程（端口 4000）
npm run start:worker
# 或：cd packages/worker && npm start

# 启动 Admin Web UI（端口 3001）
npm run start:admin
# 或：cd packages/admin-web && npm start

# 启动 CRM PC IM（Electron + Vite 开发服务器）
cd packages/crm-pc-im && npm run dev

# 并发启动所有服务
npm run dev        # Master + Worker
npm run dev:all    # Master + Worker + Admin
```

### 测试

```bash
# 运行所有工作区的测试
npm test

# 测试特定 package
npm run test --workspace=packages/master
npm run test --workspace=packages/worker
npm run test --workspace=packages/shared

# 或进入 package 目录
cd packages/master && npm test
cd packages/worker && npm test

# 开发时的监听模式
cd packages/master && npm test -- --watch

# 生成覆盖率报告
cd packages/master && npm test -- --coverage
```

### 数据库管理

```bash
# 清理/重置 Master 数据库
cd packages/master && npm run clean:db
```

### 生产部署

```bash
# 构建生产版本
npm run build

# 使用 PM2 部署
pm2 start packages/master/src/index.js --name hiscrm-master
pm2 start packages/worker/src/index.js --name hiscrm-worker-1 -- --worker-id worker-1 --port 4001

# 多个 Worker
pm2 start packages/worker/src/index.js --name hiscrm-worker-2 -- --worker-id worker-2 --port 4002
pm2 start packages/worker/src/index.js --name hiscrm-worker-3 -- --worker-id worker-3 --port 4003

# 监控
pm2 monit
pm2 logs
```

## 代码架构

### 通信协议

所有 Master-Worker 和 Master-Client 通信使用 Socket.IO，预定义的消息类型在 `packages/shared/protocol/messages.js` 中：

- **Worker → Master**：`WORKER_REGISTER`、`WORKER_HEARTBEAT`、`WORKER_MESSAGE_DETECTED`、`WORKER_ACCOUNT_STATUS`
- **Master → Worker**：`MASTER_TASK_ASSIGN`、`MASTER_TASK_REVOKE`、`MASTER_ACCOUNT_LOGOUT`
- **Client ↔ Master**：`CLIENT_CONNECT`、`CLIENT_SYNC_REQUEST`、`MASTER_NOTIFICATION_PUSH`

使用 `createMessage()` 辅助函数创建格式正确的消息。

### Master 服务器结构

```
packages/master/src/
├── index.js                    # 入口文件，初始化所有组件
├── api/routes/                 # HTTP REST API 端点
├── communication/              # Socket.IO 服务器和消息处理器
│   ├── socket-server.js        # Socket.IO 初始化
│   ├── message-receiver.js     # 处理 Worker 消息
│   └── notification-broadcaster.js
├── database/                   # 数据访问层
│   ├── init.js                 # 数据库初始化
│   ├── schema.sql              # 最终 schema（v1.0，无迁移）
│   ├── *-dao.js                # 每个表的数据访问对象
│   └── schema-validator.js     # Schema 完整性验证
├── worker_manager/             # Worker 生命周期管理
│   ├── registration.js         # Worker 注册
│   ├── account-assigner.js     # 将账户分配给 Worker
│   └── account-status-updater.js
├── scheduler/                  # 任务调度
├── monitor/                    # 心跳监控
└── login/                      # 二维码登录协调
```

**重要**：数据库使用最终 schema 方案（无迁移）。`schema.sql` 文件代表当前状态（v1.0）。如果需要修改 schema，直接更新 `schema.sql`，并使用 `schema-validator.js` 验证完整性。

### Worker 进程结构

```
packages/worker/src/
├── index.js                    # 入口文件
├── platforms/                  # 平台特定实现
│   ├── base/
│   │   ├── platform-base.js    # 平台的抽象基类
│   │   └── worker-bridge.js    # Worker 与 Platform 之间的桥接
│   ├── douyin/                 # 抖音（中国版 TikTok）实现
│   │   ├── platform.js         # 主平台类
│   │   ├── crawl-comments.js
│   │   ├── crawl-direct-messages-v2.js
│   │   └── send-reply-*.js     # 回复功能
│   └── xiaohongshu/            # 小红书平台
├── browser/                    # 多浏览器隔离管理器
├── handlers/                   # 任务执行处理器
│   ├── task-runner.js
│   ├── account-initializer.js
│   └── account-status-reporter.js
├── communication/              # Socket.IO 客户端
│   ├── socket-client.js
│   ├── registration.js
│   └── heartbeat.js
├── services/                   # 支持服务
│   └── cache-manager.js        # 本地 SQLite 缓存
└── debug/                      # 调试工具
    └── chrome-devtools-mcp.js  # Chrome DevTools 集成
```

**核心概念 - 多浏览器架构**：每个账户都有独立的浏览器进程，包含：
- 用户数据目录：`./data/browser/{worker-id}/browser_{account-id}/`
- 指纹配置：`./data/browser/{worker-id}/fingerprints/{account-id}_fingerprint.json`
- 存储状态（cookies）：`./data/browser/{worker-id}/storage-states/{account-id}_storage.json`

内存使用：每个账户约 200MB。推荐：每个 Worker ≤10 个账户。

### 平台系统

Worker 使用插件架构支持多平台。添加新平台的步骤：

1. 继承 `PlatformBase` 类（`packages/worker/src/platforms/base/platform-base.js`）
2. 实现必需方法：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liuhongtao1981/HISCRM-IM-Portal](https://github.com/liuhongtao1981/HISCRM-IM-Portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
