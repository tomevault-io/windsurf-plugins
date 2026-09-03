---
trigger: always_on
description: > 本指南供 AI coding agent 使用。人类开发者请优先参考 `README.md` 与相关 `docs/`。
---

# AGENTS.md - Wemux 项目开发指南

> 本指南供 AI coding agent 使用。人类开发者请优先参考 `README.md` 与相关 `docs/`。

## Open Core 边界

- 本仓库是公开核心，只能提交社区版代码和明确允许公开的文档。
- 可选的商业扩展通过 gate/registry 边界装配；不得把商业实现或内部运营材料复制进本仓库。
- `docs/` 只收录面向社区的开发、自托管、架构和贡献文档；内部战略、发布、营销、调研、生产运维和实施方案不得加入。
- 修改或推送前运行 `node scripts/open-core/public-boundary.mjs --staged`；
  公开构建必须在没有可选扩展挂载的干净环境中通过。

## 核心哲学

**1. 好品味优先**
- 优先消灭特殊分支，而不是堆更多 `if`
- 能通过重构让边界自然消失，就不要补丁式修补
- 复杂度会上瘾，默认先怀疑复杂方案

**2. Worker-first**
- Wemux 是一个“由 worker 执行代码任务”的平台
- `server` 是控制面，不是本地代码执行器
- `web` 是控制台，不是执行环境
- 涉及代码执行、worktree、仓库准备、agent runtime 时，默认先看 `worker`

**3. 实用主义**
- 优先解决真实线上/开发流问题
- 不为假想抽象重写整条链路
- 小步修正，保持系统可运行、可验证、可回退

**4. 简洁执念**
- 函数应短小、单一职责
- 超过 3 层缩进时，优先重构
- 新增函数尽量控制在 100 行内
- 新增文件若逼近 800 行，优先拆分，至少不要继续恶化

## 项目现实

- 当前仓库是 monorepo，不再是早期的 `src/` + `server/` 扁平结构
- 前端在 `apps/web`
- 控制面后端在 `apps/server`
- 执行器 / daemon / 本地控制台在 `apps/worker`
- 跨端共享类型与契约在 `packages/shared`
- Postgres 是主持久化存储；**schema / 迁移由 Drizzle 管理**（见下方「Postgres 与 Drizzle」）
- S3 兼容对象存储用于头像、图片等非代码文件对象；任务 patch 与测试文件不得上传到控制面
- worker 是唯一代码执行入口；没有在线 worker 时，不应偷偷回退成 server 本地执行
- Main Chat、Direct Chat、Group Chat 与外部渠道 Agent 涉及项目仓库的代码、Git、测试与构建时，优先派发到 Workspace Session 执行，以获得正确仓库上下文、隔离 worktree 与可追溯记录；不涉及项目仓库，或没有关联项目/可用 Workspace 时，允许在 Agent 自己的默认工作目录直接完成。这是提示词层的优先级约定，运行时不再按渠道限制工具能力
- 普通聊天或外部渠道中的项目代码工作默认先创建 Task，再选择或创建 Workspace 并派发执行；只有用户在当前消息中明确要求直接创建工作区时才允许跳过 Task
- Agent 都是用户拥有的普通 Agent；`CEO Agent` 只是一份新用户首 Agent 示例模板，创建后可自由修改或删除，不具备系统身份且删除后不会自动重建

## 当前目录结构

```text
.
├── apps/
│   ├── web/                    # React + Vite + TanStack Start
│   │   ├── src/routes/         # 页面路由
│   │   └── src/components/     # UI 与业务组件
│   ├── server/                 # Hono 控制面 / WS / 调度 / 集成
│   │   └── src/
│   ├── worker/                 # 本地执行器 daemon / worker console
│   │   └── src/
│   ├── desktop/                # Electron 桌面客户端
│   ├── mobile/                 # React Native + Expo 移动客户端
├── packages/
│   └── shared/                 # 共享类型、契约、工具
├── scripts/                    # 构建与开发脚本
├── docs/                       # 面向社区的开发、架构与部署文档
└── dist-server/                # 服务端构建产物
```

## 页面与会话边界

### 三个页面概念必须分清

- `http://app.wemux.localtest.me:15173/chat` 是 **Agent Chat / 主聊天页**
- `http://app.wemux.localtest.me:15173/workspace?...` 是 **单个工作区详情页**
- `http://app.wemux.localtest.me:15173/workspaces?...` 是 **工作区列表 / 工作区会话入口页**

### 默认语义

- 用户说“AI 会话”“聊天窗口”“agent chat”“对话页”时，默认指 `/chat`
- `/chat` 同时支持直接 Agent 会话和组织群聊；群聊只是人/Agent 沟通容器，不等于任务负责人中的 Squad，也不进入任务指派或评论 Mention 目录
- 用户说“工作区”“工作台”“workspace detail”“单个会话工作区”时，默认指 `/workspace`
- 用户说“工作区会话列表”“workspaces 页面”“某任务下有哪些会话”时，默认指 `/workspaces`

### 禁止混用

- 不得把 `/chat` 当作 workspace session 页面处理
- 不得把 `/workspace` 当作主 chat 页面处理
- 不得把 `/workspaces` 当作单个 workspace detail 页面处理
- 用户已经明确说“这是 agent chat”或“这是工作区会话”时，后续实现必须严格服从

### 动手前先定位代码

- `/chat` 先看 `apps/web/src/routes/chat.tsx` 与 `apps/web/src/routes/-chat-route/*`
- `/workspace` 先看 `apps/web/src/routes/workspace.tsx` 与 `components/workspaces/*`
- `/workspaces` 先看 `apps/web/src/routes/workspaces.tsx` 与 `components/workspaces/*`
- server 侧主聊天相关优先看 `project-main-chat*`、`conversation-routes.ts`
- server 侧工作区相关优先看 `collaboration-workspace-routes.ts`、`workspace-group-chat-routes.ts`、`workspace-management-routes.ts`

### 命名要求

- 变量、函数、日志、注释里尽量写清 `mainChatSession`、`workspaceSession`、`distributedTask`
- 避免只写含糊的 `session`、`chatData`、`workspaceData`
- 只要跨页面共享逻辑，名称里要反映作用域，避免后续 AI 再改错地方

## 架构职责

### `apps/web`

- 负责 UI、路由、交互状态、表单、展示与调用控制面 API
- 不要在 web 里发明本地执行逻辑
- 路由相关逻辑优先靠近 `routes/` 与对应 `components/` 放置

### `apps/server`

- 负责 Hono HTTP / WebSocket、调度、鉴权、计费、聊天编排、工作区/任务控制面
- server 可以做 orchestration、state aggregation、artifact registration
- server 不应承担 worker 本地仓库执行职责

### `apps/worker`

- 负责配对、daemon、仓库准备、worktree、agent CLI/runtime、产物回传
- 任何 Git worktree、仓库 checkout、patch 生成、终端执行问题，默认先从 `worker` 查
- 修改执行链路时，要小心控制面协议兼容

### `packages/shared`

- 放共享类型、协议、数据结构、跨端纯函数
- 只要 web/server/worker 至少两端都要理解的结构，优先放这里
- 禁止在三端各自复制一份相同类型再慢慢漂移

## Worker 本地存储与多用户隔离

worker 本地目录要按“节点级、用户私有、workspace 共享”分层，长期目标结构见 `docs/WORKER-LOCAL-STORAGE.md`：

```text
~/.wemux-dev/
├── node/
│   ├── config.json
│   ├── machine-id
│   ├── runtime/
│   └── cache/
├── users/
│   └── <userId>/
│       ├── projects/
│       ├── repos/
│       ├── worktrees/
│       ├── runtime/
│       └── cache/
└── workspaces/
    └── <workspaceId>/
        ├── projects/
        ├── repos/
        ├── worktrees/
        ├── cache/
        └── artifacts/
```

目录规则：

- `node/` 只放机器级配置、machine id、节点级 daemon/tool runtime/cache，不放项目代码或用户凭据。
- 私人项目或尚未进入执行 workspace 的资源落在 `users/<ownerUserId>/projects|repos|worktrees`。
- Codex/Claude 等带凭据的 agent runtime 落在 `users/<actingUserId>/runtime`，不能放进 workspace 共享目录。
- workspace session 执行资源落在 `workspaces/<workspaceId>/projects|repos|worktrees|cache|artifacts`。
- 同一个 workspace 被多个成员使用时，路径只使用真实 `workspaceId`；不要按 owner 或 acting user 再拆一份 workspace 目录。
- `project.workspaceId` 是项目可见性/团队归属，不等于本地执行 workspace id；只有真实 `WorkspaceRecord.id` / workspace session scope 才能用于本地路径。
- `workspaceSessionId` 不能替代 `workspaceId` 出现在目录层级里。
- `unknown` 只能作为历史路径识别或迁移信号，不能作为新建目录的 userId 或 workspaceId。

改本地路径相关代码时：

- 优先改 `packages/shared/src/workspace-paths.ts` 里的共享 helper，再同步消费方。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wemux-ai/wemux](https://github.com/wemux-ai/wemux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
