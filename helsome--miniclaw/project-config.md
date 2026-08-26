---
trigger: always_on
description: 本文档描述当前代码的工程约束和导航入口，供人类与 AI 协作者共同使用。
---

# Miniclaw — 工程协作指南

本文档描述当前代码的工程约束和导航入口，供人类与 AI 协作者共同使用。

## 1. 文档与代码真相源

- 产品介绍、安装和常用操作：`README.md`
- 路由族与主要 Web API：`docs/API.md`
- 权限边界：`docs/ACL-MATRIX.md`
- 运行时 Prompt：`container/agent-runner/prompts/`
- 数据库 Schema：`src/db.ts` 中的 `CURRENT_SCHEMA_VERSION` 与建表/迁移代码
- Web 路由：`web/src/App.tsx`
- 系统设置及默认值：`src/runtime-config.ts`
- 渠道能力与会话路由：`src/im-channel-capabilities.ts`、`src/channel-mount-service.ts`
- StreamEvent：`shared/stream-event.ts`

`docs/agent-first-architecture-plan.md` 和
`docs/claude-code-plugin-automation-design.md` 是历史设计记录，不作为当前接口或数据结构的真相源。

## 2. 产品模型

Miniclaw 是基于 Pi Agent Runtime 的自托管、多用户 Agent 工作台，支持 Web 与飞书、
Telegram、QQ、钉钉、微信、Discord、WhatsApp。

当前产品层级：

```text
Agent Profile（身份、四段 Prompt、能力策略）
└── Workspace（文件目录、执行模式、环境变量、渠道群聊绑定）
    ├── Main Session
    ├── Runtime Session（独立 Pi 上下文，可绑定私聊）
    ├── Native Context Session（飞书话题等原生线程）
    └── Scheduled Run（group 或 isolated）
```

重要命名边界：

- `agent_profiles` 是产品级 Agent。
- `registered_groups` 是当前兼容层中的工作区和渠道路由记录。
- `agents` 表及 `/api/groups/:jid/agents` 是历史命名，实际表示工作区内的运行会话，
  不是产品级 Agent Profile。
- 同一个 Workspace 内的会话共享工作区文件目录，但拥有独立 Pi Session；
  工作区文件隔离与对话上下文隔离是两件事。

## 3. 主要模块

### 3.1 主服务

| 文件/目录                          | 职责                                                   |
| ---------------------------------- | ------------------------------------------------------ |
| `src/index.ts`                     | 启动、消息消费、渠道路由、IPC、调度与 Agent 运行编排   |
| `src/web.ts`                       | Hono 应用、路由挂载、Cookie 认证、WebSocket 与静态资源 |
| `src/db.ts`                        | SQLite Schema、迁移和持久化访问器                      |
| `src/group-queue.ts`               | Session 串行、Runner 生命周期、重试与容量控制          |
| `src/container-runner.ts`          | Host/Container Runner、挂载、环境与能力快照            |
| `src/task-scheduler.ts`            | Cron、interval、once 调度和重启恢复                    |
| `src/channel-mount-service.ts`     | 工作区/会话绑定和原生线程路由                          |
| `src/channel-reliability-store.ts` | Inbox、Turn、Outbox、Streaming Card 的持久状态机       |
| `src/im-manager.ts`                | 多用户、多账号渠道连接池                               |
| `src/agent-capability-preview.ts`  | Agent 最终上下文和能力预览                             |
| `src/claude-context-resolver.ts`   | 兼容上下文、Skills 与来源解析                         |

渠道实现位于：

- `src/feishu.ts`
- `src/telegram.ts`
- `src/qq.ts`
- `src/dingtalk.ts`
- `src/wechat.ts`
- `src/discord.ts`
- `src/whatsapp.ts`

HTTP 路由位于 `src/routes/`，完整模块索引见 `docs/API.md`。

### 3.2 Web

Web 位于 `web/`，使用 React 19、Vite、Tailwind CSS 4、React Router、Zustand 和
Radix UI。路由以 `web/src/App.tsx` 为准：

| 路径                      | 用途                                  |
| ------------------------- | ------------------------------------- |
| `/setup`                  | 首个管理员初始化                      |
| `/setup/providers`        | Provider 引导                         |
| `/setup/channels`         | 用户渠道引导                          |
| `/login`、`/register`     | 登录和注册                            |
| `/chat/:groupFolder?`     | 工作台与会话                          |
| `/agent-profiles`         | Agent 管理                            |
| `/capabilities/:section?` | Skills、MCP、Plugins                  |
| `/tasks`                  | 定时任务                              |
| `/usage`、`/billing`      | 用量与计费                            |
| `/memory`                 | 记忆管理                              |
| `/settings`               | 账户和系统设置                        |
| `/monitor`                | 运行状态，需要 `manage_system_config` |
| `/users`                  | 用户、邀请和审计管理                  |

`/groups`、`/skills`、`/mcp-servers` 和 `/plugins` 是兼容重定向，不应新增独立页面。

### 3.3 Agent Runner

`container/agent-runner/` 同时服务 Host 和 Container 两种执行模式：

- stdin 接收 `ContainerInput`。
- stdout 使用 `OUTPUT_START_MARKER` / `OUTPUT_END_MARKER` 输出结构化结果。
- 后续消息、工具请求和关闭控制通过独立 IPC 目录传递。
- `container/agent-runner/prompts/` 中的 Prompt 在启动时加载。
- Miniclaw MCP 工具由 `container/agent-runner/src/mcp-tools.ts` 注册。
- `shared/stream-event.ts` 同步到主服务、Web 和 Runner。

不要在文档中维护固定的 MCP 工具数量或 StreamEvent 数量；它们会随能力演进变化，
应直接查看类型与注册代码。

## 4. 执行和并发

| 模式      | 行为                                                          | 容量边界                                             |
| --------- | ------------------------------------------------------------- | ---------------------------------------------------- |
| Host      | Runner 作为宿主机 Node 进程运行，`customCwd` 直接作为工作目录 | 同一 Session 串行；不同 Session 不设置应用层并发上限 |
| Container | Runner 在非 root Docker 容器运行，通过只读/读写挂载访问资源   | 受 `maxConcurrentContainers` 和用户计费配额限制      |

共同约束：

- 同一序列化键内的消息保持顺序。
- 不同飞书话题、不同 Runtime Session 使用不同序列化键，可以并发。
- 普通消息与定时任务使用明确的队列状态；失败采用有界指数退避。
- `CONTAINER_TIMEOUT` 控制单次运行上限，`IDLE_TIMEOUT` 控制暖 Runner 的空闲保留时间。
- Script 任务与其他任务共用 `CONTAINER_TIMEOUT`，不设置独立并发池或超时配置。
- 服务重启时，错过的周期任务记为 `missed` 并推进计划；一次性任务仍会补跑。

Host 模式没有 `maxConcurrentHostProcesses`。旧客户端提交该字段时后端仅为兼容而忽略，
不得重新把它实现为全局 Host 并发池。

## 5. Agent Prompt 与能力

自定义 Agent 使用四段 Prompt：

- `IDENTITY`：身份、使命和边界。
- `SOUL`：稳定价值观、判断原则和表达风格，可为空。
- `AGENTS`：工作流、输入输出、默认值、分支和失败处理。
- `TOOLS`：Skill、MCP 和工具的选择方式与限制，可为空。

运行时能力不是简单拼接文本：

1. `claude-context-resolver` 解析 managed 或 `host_claude` 兼容上下文。
2. Effective Skill/MCP Resolver 生成精确清单与 hash。
3. Container 模式逐个只读挂载选中的 Skill。
4. Host 模式也使用同步后的 Session `.claude` 目录和相同能力清单。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [helsome/miniclaw](https://github.com/helsome/miniclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
