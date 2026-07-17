---
trigger: always_on
description: 本文档帮助 AI 和工程协作者快速理解项目架构、关键机制与修改边界。
---

# DeepThink — AI 协作者指南

本文档帮助 AI 和工程协作者快速理解项目架构、关键机制与修改边界。

## 1. 项目定位

DeepThink, 企业级自主 Agent 超级智能体自进化平台，从 Harness Engineering 到 Loop Engineering 范式的先行者，是面向企业客户的新一代 AI 基础设施(AI Infra)。DeepThink 平台以多 Agent 协作框架为核心，融合 AI 自主编程（AI Coding）、自主进化（Self-Evolving）、全栈可观测性（Full-Stack Observability）、Bug 自修复闭环（Bug Auto-Fix Loop） 与 程序员-Agent 共生协作（Human-Agent Symbiosis），构建一个能持续学习、自我改进、最终成长为超级智能体的企业级 AI 系统：
• AI 自主研发平台——Agent 独立完成软件研发全生命周期，无需人类工程师介入常规编码任务；
• 自进化智能体引擎——Agent 持续从错误中学习、从代码库中吸收知识、从用户反馈中进化；
• 程序员-Agent 协作中枢——每位程序员拥有个人"开发项目"，内含多个并行会话，中央调度防止并发冲突；
• 企业级 SaaS 平台——多租户隔离、权限分级、计费弹性、企业集成（飞书/钉钉/企微/LDAP）；
• 超级智能体孵化器——通过持续进化，单一 Agent 最终具备完整软件团队综合能力。

> "让每一家企业都拥有一支永不停歇、持续进化的 AI 超级研发团队——从工具使用者，到代码创造者，最终成长为可自我繁衍的超级智能体。让我们在通往 AGI 的道路上一起前行。"



## 2. 核心架构

### 2.1 后端模块

| 模块 | 职责 |
|------|------|
| `src/index.ts` | 入口：管理员引导、消息轮询（2s）、IPC 监听（1s）、容器生命周期 |
| `src/web.ts` | Hono 框架：路由挂载、WebSocket 升级、HMAC Cookie 认证、静态文件托管 |
| `src/routes/auth.ts` | 认证：登录 / 登出 / 注册、`GET /api/auth/me`（含 `setupStatus`）、设置向导、RBAC、邀请码 |
| `src/routes/groups.ts` | 群组 CRUD、消息分页、会话重置（重建工作区）、群组级容器环境变量 |
| `src/routes/files.ts` | 文件上传（默认 50MB 限制，见 `MAX_FILE_SIZE_MB`）/ 下载 / 删除、目录管理、路径遍历防护 |
| `src/routes/config.ts` | Claude / 飞书配置（AES-256-GCM 加密存储）、连通性测试、批量应用到所有容器、per-user IM 通道配置（`/api/config/user-im/feishu`、`/api/config/user-im/telegram`、`/api/config/user-im/qq`、`/api/config/user-im/dingtalk`、`/api/config/user-im/whatsapp`） |
| `src/routes/monitor.ts` | 系统状态：容器列表、队列状态、健康检查（`GET /api/health` 无需认证） |
| `src/routes/memory.ts` | 记忆文件读写（`groups/global/` + `groups/{folder}/`）、全文检索 |
| `src/routes/tasks.ts` | 定时任务 CRUD + 执行日志查询 |
| `src/routes/skills.ts` | Skills 列表与管理 |
| `src/routes/admin.ts` | 用户管理、邀请码、审计日志、注册设置 |
| `src/routes/browse.ts` | 目录浏览 API（`GET/POST /api/browse/directories`，受挂载白名单约束） |
| `src/routes/agents.ts` | Sub-Agent CRUD（`GET/POST/DELETE /api/groups/:jid/agents`） |
| `src/routes/mcp-servers.ts` | MCP Servers 管理（CRUD + `POST /api/mcp-servers/sync-host`，per-user） |
| `src/routes/plugins.ts` | Claude Code Plugins 管理（catalog + per-user enable + versioned runtime）：admin 通过 `POST /api/plugins/catalog/scan` 触发宿主机扫描共享导入 catalog；用户从 catalog enable（`PATCH /api/plugins/enabled/:fullId`，自动 materialize runtime）；`DELETE /api/plugins/marketplaces/:name` 仅清除调用者自己的 enabled refs，不删 catalog |
| `src/plugin-utils.ts` | Plugin 加载工具：`loadUserPlugins(userId, {runtime})` → `SdkPluginConfig[]`；per-user enable refs 在 `data/plugins/users/{userId}/plugins.json`，runtime materialize 到 `data/plugins/runtime/{userId}/snapshots/{snapshotId}/{marketplace}/{plugin}/` |
| `src/plugin-dependency-check.ts` | Plugin 依赖 best-effort 预检：扫描 plugin 目录下 `commands/*.md` frontmatter 的 `allowed-tools: Bash()` + `hooks/hooks.json` 的 command 第一 token；`config/plugin-deps-override.json` 人工覆盖表优先级最高 |
| `src/feishu.ts` | 飞书连接工厂（`createFeishuConnection`）：WebSocket 长连接、消息去重（LRU 1000 条 / 30min TTL）、富文本卡片、Reaction；`file` 消息下载到工作区；`post` 图文消息仅提取文字 |
| `src/telegram.ts` | Telegram 连接工厂（`createTelegramConnection`）：Bot API Long Polling、Markdown → HTML 转换、长消息分片（3800 字符）；`message:photo` 下载为 base64 供 Vision；`message:document` 下载文件到工作区 |
| `src/qq.ts` | QQ 连接工厂（`createQQConnection`）：Bot API v2 WebSocket 长连接、OAuth Token 管理、C2C 私聊 + 群聊 @Bot、消息去重（LRU 1000 条 / 30min TTL）、Markdown → 纯文本、长消息分片（5000 字符）、图片下载为 base64 供 Vision |
| `src/dingtalk.ts` | 钉钉连接工厂（`createDingTalkConnection`）：Stream 协议长连接、消息去重（LRU 1000 条 / 30min TTL）；支持 `text`、`picture`（通过 downloadCode 下载）和 `image`（通过 contentUrl 下载）；图片超过 5MB 不发 base64，仅保存到 `downloads/dingtalk/` |
| `src/whatsapp.ts` | WhatsApp 连接工厂（`createWhatsAppConnection`）：基于 `@whiskeysockets/baileys` 的 WhatsApp Web 协议；`useMultiFileAuthState` 持久化登录态；QR 经 `qrcode` 渲染 PNG data URL 推前端；自动 3s 重连（logged_out 不重连避免 QR 风暴）；`messages.upsert` 转发文本 + 媒体（image/video/audio/document）下载到 `downloads/whatsapp/{date}/`；小图片附 base64 attachment 供 Vision；`group-participants.update` 触发 onBotAddedToGroup / onBotRemovedFromGroup；群组 `require_mention` 通过 `mentionedJid` 与 `sock.user.id` 比对。详见 [`docs/channels/whatsapp.md`](docs/channels/whatsapp.md) |
| `src/dingtalk-streaming-card.ts` | 钉钉 AI Card 流式响应控制器（打字机效果） |
| `src/im-downloader.ts` | IM 文件下载工具：`saveDownloadedFile()` 将 Buffer 写入 `downloads/{channel}/{YYYY-MM-DD}/`，支持 `feishu`/`telegram`/`qq`/`dingtalk` 通道，处理路径安全、文件名冲突和大小限制（默认 50MB，见 `MAX_FILE_SIZE_MB`） |
| `src/im-manager.ts` | IM 连接池管理器（`IMConnectionManager`）：per-user 飞书/Telegram/QQ/钉钉/Discord/WhatsApp 连接管理、热重连、批量断开 |
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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AIGeniusInstitute/deepthink](https://github.com/AIGeniusInstitute/deepthink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
