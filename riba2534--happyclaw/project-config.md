---
trigger: always_on
description: 本文档帮助 AI 和工程协作者快速理解项目架构、关键机制与修改边界。
---

# HappyClaw — AI 协作者指南

本文档帮助 AI 和工程协作者快速理解项目架构、关键机制与修改边界。

## 1. 项目定位

HappyClaw 是一个自托管的多用户 AI Agent 系统：

- **输入**：飞书 / Telegram / QQ / 钉钉 / Web 界面消息（每个用户可独立配置 IM 通道）
- **执行**：Docker 容器或宿主机进程中运行 Claude Agent（基于 Claude Agent SDK），每个用户拥有独立主容器
- **输出**：飞书富文本卡片 / Telegram HTML / QQ 纯文本 / Web 实时流式推送
- **记忆**：Agent 自主维护 `CLAUDE.md` 和工作区文件，实现跨会话持久记忆

## 2. 核心架构

### 2.1 后端模块

| 模块 | 职责 |
|------|------|
| `src/index.ts` | 入口：管理员引导、消息轮询（2s）、IPC 监听（1s）、容器生命周期 |
| `src/web.ts` | Hono 框架：路由挂载、WebSocket 升级、HMAC Cookie 认证、静态文件托管 |
| `src/routes/auth.ts` | 认证：登录 / 登出 / 注册、`GET /api/auth/me`（含 `setupStatus`）、设置向导、RBAC、邀请码 |
| `src/routes/groups.ts` | 群组 CRUD、消息分页、会话重置（重建工作区）、群组级容器环境变量 |
| `src/routes/files.ts` | 文件上传（50MB 限制）/ 下载 / 删除、目录管理、路径遍历防护 |
| `src/routes/config.ts` | Claude / 飞书配置（AES-256-GCM 加密存储）、连通性测试、批量应用到所有容器、per-user IM 通道配置（`/api/config/user-im/feishu`、`/api/config/user-im/telegram`、`/api/config/user-im/qq`、`/api/config/user-im/dingtalk`） |
| `src/routes/monitor.ts` | 系统状态：容器列表、队列状态、健康检查（`GET /api/health` 无需认证） |
| `src/routes/memory.ts` | 记忆文件读写（`groups/global/` + `groups/{folder}/`）、全文检索 |
| `src/routes/tasks.ts` | 定时任务 CRUD + 执行日志查询 |
| `src/routes/skills.ts` | Skills 列表与管理 |
| `src/routes/admin.ts` | 用户管理、邀请码、审计日志、注册设置 |
| `src/routes/browse.ts` | 目录浏览 API（`GET/POST /api/browse/directories`，受挂载白名单约束） |
| `src/routes/agents.ts` | Sub-Agent CRUD（`GET/POST/DELETE /api/groups/:jid/agents`） |
| `src/routes/mcp-servers.ts` | MCP Servers 管理（CRUD + `POST /api/mcp-servers/sync-host`，per-user） |
| `src/feishu.ts` | 飞书连接工厂（`createFeishuConnection`）：WebSocket 长连接、消息去重（LRU 1000 条 / 30min TTL）、富文本卡片、Reaction；`file` 消息下载到工作区；`post` 图文消息仅提取文字 |
| `src/telegram.ts` | Telegram 连接工厂（`createTelegramConnection`）：Bot API Long Polling、Markdown → HTML 转换、长消息分片（3800 字符）；`message:photo` 下载为 base64 供 Vision；`message:document` 下载文件到工作区 |
| `src/qq.ts` | QQ 连接工厂（`createQQConnection`）：Bot API v2 WebSocket 长连接、OAuth Token 管理、C2C 私聊 + 群聊 @Bot、消息去重（LRU 1000 条 / 30min TTL）、Markdown → 纯文本、长消息分片（5000 字符）、图片下载为 base64 供 Vision |
| `src/dingtalk.ts` | 钉钉连接工厂（`createDingTalkConnection`）：Stream 协议长连接、消息去重（LRU 1000 条 / 30min TTL）；支持 `text`、`picture`（通过 downloadCode 下载）和 `image`（通过 contentUrl 下载）；图片超过 5MB 不发 base64，仅保存到 `downloads/dingtalk/` |
| `src/dingtalk-streaming-card.ts` | 钉钉 AI Card 流式响应控制器（打字机效果） |
| `src/im-downloader.ts` | IM 文件下载工具：`saveDownloadedFile()` 将 Buffer 写入 `downloads/{channel}/{YYYY-MM-DD}/`，支持 `feishu`/`telegram`/`qq`/`dingtalk` 通道，处理路径安全、文件名冲突和 50MB 限制 |
| `src/im-manager.ts` | IM 连接池管理器（`IMConnectionManager`）：per-user 飞书/Telegram/QQ/钉钉连接管理、热重连、批量断开 |
| `src/container-runner.ts` | 容器生命周期：Docker run + 宿主机进程模式、卷挂载构建（isAdminHome 区分权限）、环境变量注入 |
| `src/agent-output-parser.ts` | Agent 输出解析：OUTPUT_MARKER 流式输出解析、stdout/stderr 处理、进程生命周期回调（从 container-runner.ts 提取的共享逻辑） |
| `src/group-queue.ts` | 并发控制：最大 20 容器 + 最大 5 宿主机进程、会话级队列、任务优先于消息、指数退避重试 |
| `src/runtime-config.ts` | 配置存储：AES-256-GCM 加密、分层配置（容器级 > 全局 > 环境变量）、变更审计日志 |
| `src/task-scheduler.ts` | 定时调度：60s 轮询、cron / interval / once 三种模式、group / isolated 上下文 |
| `src/file-manager.ts` | 文件安全：路径遍历防护、符号链接检测、系统路径保护（`logs/`、`CLAUDE.md`、`.claude/`、`conversations/`） |
| `src/mount-security.ts` | 挂载安全：白名单校验、黑名单模式匹配（`.ssh`、`.gnupg` 等）、非主会话只读强制 |
| `src/db.ts` | 数据层：SQLite WAL 模式、Schema 版本校验（v1→v24）、核心表定义 |
| `src/auth.ts` | 密码工具：bcrypt 哈希/验证、Session Token 生成、用户名/密码校验 |
| `src/permissions.ts` | 权限常量和模板定义（`ALL_PERMISSIONS`、`PERMISSION_TEMPLATES`） |
| `src/schemas.ts` | Zod v4 校验 schema：API 请求体校验 |
| `src/utils.ts` | 工具函数：`getClientIp()`（TRUST_PROXY 感知） |
| `src/web-context.ts` | Web 共享状态：`WebDeps` 依赖注入、群组访问权限检查、WS 客户端管理 |
| `src/middleware/auth.ts` | 认证中间件：Cookie Session 校验、权限检查中间件工厂 |
| `src/channel-prefixes.ts` | IM channel type → JID prefix 映射，被多处共享；**新增渠道时必须同步更新此文件** |
| `src/im-channel.ts` | 统一 IM 通道接口（`IMChannel`）、Feishu/Telegram 适配器工厂 |
| `src/commands.ts` | Web 端斜杠命令处理器（`/clear` 重置会话） |
| `src/im-command-utils.ts` | IM 斜杠命令纯函数工具：`formatWorkspaceList()`、`formatContextMessages()` |
| `src/telegram-pairing.ts` | Telegram 配对码：6 位随机码，5 分钟过期 |
| `src/terminal-manager.ts` | Docker 容器终端管理（node-pty + pipe fallback，WebSocket 双向通信） |
| `src/message-attachments.ts` | 图片附件规范化（MIME 检测、Data URL 解析） |
| `src/image-detector.ts` | 图片 MIME 检测（magic bytes），由 `shared/image-detector.ts` 同步 |
| `src/daily-summary.ts` | 每日对话汇总（凌晨 2-3 点，per-user，写入 HEARTBEAT.md） |
| `src/script-runner.ts` | 脚本任务执行器（`exec()` + 并发限制 + 超时 + 1MB 输出缓冲） |
| `src/reset-admin.ts` | 管理员密码重置脚本入口 |
| `src/config.ts` | 常量：路径、超时、并发限制、会话密钥（优先级：环境变量 > 文件 > 生成，0600 权限） |
| `src/logger.ts` | 日志：pino + pino-pretty |

### 2.2 前端

| 层次 | 技术 |
|------|------|
| 框架 | React 19 + TypeScript + Vite 6 |
| 状态 | Zustand 5（10 个 Store：auth、chat、groups、tasks、monitor、container-env、files、users、skills、mcp-servers） |
| 样式 | Tailwind CSS 4（teal 主色调，`lg:` 断点响应式，移动端优先） |
| 路由 | React Router 7（AuthGuard + SetupPage 重定向） |
| 通信 | 统一 API 客户端（8s 超时，FormData 120s）、WebSocket 实时推送 + 指数退避重连 |
| 渲染 | react-markdown + remark-gfm + rehype-highlight（代码高亮）、mermaid（图表渲染）、@tanstack/react-virtual（虚拟滚动） |
| UI 组件 | radix-ui + lucide-react |
| PWA | vite-plugin-pwa（生产构建始终启用，开发模式通过 `VITE_PWA_DEV=true` 启用） |

#### 前端路由表

| 路径 | 页面 | 权限 |
|------|------|------|
| `/setup` | `SetupPage` — 管理员创建向导 | 公开（仅未初始化时） |
| `/setup/providers` | `SetupProvidersPage` — Claude/飞书配置 | 登录后 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [riba2534/happyclaw](https://github.com/riba2534/happyclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
