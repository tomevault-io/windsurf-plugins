---
trigger: always_on
description: 本节优先级高于本文所有后续阅读、规则加载和 worktree 初始化判断。
---

# Repo Agent Guide

## 最高优先级：开发服务 Fast Path

本节优先级高于本文所有后续阅读、规则加载和 worktree 初始化判断。

如果用户意图是“取最新代码并启动 web 服务”“拉取最新代码并启动一个 web 服务”“启动 web 服务”“start web dev server”，唯一动作是在仓库根目录直接执行：

```bash
pnpm tools dev-start web
```

此类请求不要做额外预检查或成功后二次验证；`dev-start` 已经负责 fetch、安全拉取 / 对齐、workspace install 校验、端口避让、后台进程和探活。相关经验和反例见 `.oo/rules/maintenance/common-issues.md` 的 “Subagent 启动 Web 服务超过 1 分钟”。

命令退出 0 且输出 `[dev-start] ready` 时，按 target 做最短交付并停止，不要再做 `ps`、`curl` 或读日志等二次验证：

- Web / PWA / homepage / docs 等前端页面服务：用 Browser / browser-use 自动打开输出里的 `CLIENT_URL`，然后只回复一个可点击前端入口链接，例如 `[前端界面](CLIENT_URL)`；不要在成功消息里展开 `SERVER_URL`、PID 或 `LOG_FILE`，除非用户明确需要排查信息。
- Electron / desktop / launcher：直接打开输出对应的开发态应用窗口；成功消息保持简短，不要让用户再手动打开应用。
- 只有命令失败时才读取输出中的 `LOG_FILE` 并继续排查。

其他开发服务启动意图同样直接走统一 Commander CLI：

用户要求拉取最新代码并启动开发服务时，不要手动推理端口、依赖安装、后台进程或多 worktree 进程；在仓库根目录直接执行统一 Commander CLI：

```bash
pnpm tools dev-start <target>
```

意图识别表：

- 普通 Web UI / “启动一个 web 服务” / “拉取最新代码，并启动一个 web 服务”：`pnpm tools dev-start web`
- Electron / 桌面端 / launcher：`pnpm tools dev-start electron`
- Electron 且要打开当前仓库作为 workspace：`pnpm tools dev-start electron-workspace`
- PWA / 独立前端 / standalone client：`pnpm tools dev-start pwa`
- 官网首页预览 / homepage preview：`pnpm tools dev-start homepage`
- 使用文档 / docs 本地预览：`pnpm tools dev-start docs`

`pnpm tools` 通过 `scripts/run-tools.mjs` 注册 TS；如果新 worktree 缺少 register 依赖，会先执行 `pnpm install`。`dev-start` 会安全执行 `git fetch --prune origin`，在工作区干净时按当前状态拉取 / 对齐最新代码，按需校验 workspace 安装，后台启动对应开发服务，自动避开已占用端口，并在探活成功后输出 URL / PID / `LOG_FILE`。如果当前 worktree 已有同 target 服务且探活成功，会直接复用并返回 URL。

`dev-start` 会把同一 worktree 的服务状态写到 `.logs/dev-start-<target>.json`，例如 web 对应 `.logs/dev-start-web.json`。当用户询问当前目录是否已有服务、URL / PID / 端口、这个服务属于哪个 worktree，或多会话如何复用时，优先读取这个 JSON；其中 `root`、`target`、`clientUrl`、`serverUrl`、`servicePid`、`clientPid`、`serverPid`、`startedAt` 和 `managerLog` 是跨会话共享识别信息。不要为查询这些信息而另起服务；如果用户的意图是启动或拉取后启动，仍直接运行 `pnpm tools dev-start <target>`，让脚本完成复用判断和端口处理。

只有脚本失败时才继续查看输出的 `LOG_FILE`、读取更细规则或手动排查端口；不要在这些启动需求上先做冗长仓库扫描。不要使用 `screen` 管理本仓开发服务。

## 常规仓库阅读

未命中上面的开发服务 Fast Path 时，开始处理仓库前先读 `.oo/rules/` 下 `alwaysApply: true` 的基础规则与维护文档；其他文档按任务结合 `description` / `globs` 按需继续阅读。

优先阅读：

- `.oo/rules/CODING-STYLE.md`
- `.oo/rules/ARCHITECTURE.md`
- `.oo/rules/MAINTENANCE.md`

## Worktree 初始化判断

本节不适用于“取最新代码并启动 web 服务”等已命中开发服务 Fast Path 的请求；这些请求直接执行 `pnpm tools dev-start <target>`。

进入仓库后先用最小命令判断当前副本状态，再决定是否需要初始化；不要仅凭路径猜测：

- `git rev-parse --show-toplevel`：确认当前仓库根目录。
- `git status --short --branch`：确认是否 detached HEAD、是否有本地改动。
- `git log -1 --oneline --decorate` 与 `git remote -v`：确认当前提交与远端来源。
- `git worktree list`：确认当前目录是否是额外 worktree，以及主 worktree / 其他会话是否正在使用同一仓库。
- `test -d node_modules`、`test -f .oo.dev.config.json`、`test -f .env`：判断依赖与本地私有配置是否已就位。

如果当前目录是新的或刚切换来的 worktree，通常会出现这些信号：路径位于 `.codex/worktrees/` 或 `.oo/worktrees/sessions/`、`node_modules` 缺失、私有配置缺失、HEAD 处于 detached 状态，或当前提交落后于 `origin/main`。这些信号只用于决定下一步检查，不代表可以清理或覆盖文件。

## 按用户需求快速初始化

本节不适用于已命中开发服务 Fast Path 的启动请求；不要把下面的手工初始化步骤叠加到 `pnpm tools dev-start <target>` 之前。

- 只做阅读、解释、轻量检索：不安装依赖，不启动服务；读基础规则后按文件路径直接查看相关 `AGENTS.md` / 规则文档。
- 用户要求拉取最新代码：先确认工作区干净；`git fetch --prune origin` 后，如果当前是 detached HEAD 且用户没有指定分支，可对齐到 `origin/main`；如果在本地分支上，优先 `git pull --ff-only`。遇到本地改动先停下来说明，不要重置。
- 用户要求运行测试、构建、CLI、server、client 或 Electron：如果 `node_modules` 缺失，先在当前 worktree 根目录执行 `pnpm install`；如果任务依赖私有配置而 `.oo.dev.config.json` / `.env` 缺失，优先从已有本地副本确认可复用来源，不能确认时向用户说明缺口。
- 用户要求启动桌面端 / Electron：继续阅读 `apps/desktop/AGENTS.md` 与 `.oo/docs/usage/desktop.md`；如果已有正式版或其他 worktree 的 Electron 在运行，先列出 PID、启动时间、命令路径和 worktree 来源。需要并行启动开发态时，使用独立 `--user-data-dir`，避免被单实例锁转发到已有应用。
- 用户要求启动前端或调试页面：继续阅读 `.oo/rules/FRONTEND-STANDARD.md`、`.oo/rules/frontend-standard/debugging.md` 和 `apps/client/AGENTS.md`；涉及聊天页 / sender / 消息级交互时，再读 `apps/client/src/components/chat/AGENTS.md`。
- 用户要求启动后端、改 API、数据库、adapter 或 MCP：继续阅读 `.oo/rules/BACKEND-STANDARD.md`；按影响范围进入 `apps/server/src/routes/AGENTS.md`、`apps/server/src/services/*/AGENTS.md` 或相关 package 的 `AGENTS.md`。
- 用户要求改配置语义、配置页、加载 / 写回 / 分层合并：继续阅读 `.oo/rules/CONFIG.md`，再按前端或后端落点补读对应规则。
- 用户要求更新 README、接入方式、命令行为或使用说明：继续阅读 `.oo/rules/USAGE.md`，并按实际用户可见变化更新 `.oo/docs/` 公开文档内容源或对应模块 README。
- 用户要求 hooks、benchmark、发布或 changelog：分别继续阅读 `.oo/rules/HOOKS.md` / `.oo/rules/HOOKS-REFERENCE.md`、`.oo/rules/BENCHMARK.md` / `.oo/rules/BENCHMARK-PLAN.md`、`.oo/rules/RELEASE.md` 与 `changelog/`。

按任务继续阅读：

- adapter runtime / mock home / 原生资产自动适配：`.oo/rules/ADAPTERS.md`
- 配置加载、写回、分层合并或配置页 source 语义：`.oo/rules/CONFIG.md`
- 前端 / 后端约束：`.oo/rules/FRONTEND-STANDARD.md`、`.oo/rules/BACKEND-STANDARD.md`
- 桌面端 / Electron 打包、发布与本地调试：`apps/desktop/AGENTS.md`、`.oo/docs/usage/desktop.md`
- 仓库开发与贡献：`.oo/rules/DEVELOPMENT.md`
- 复杂任务拆分、子线程协作、交叉审阅或经验沉淀：`.oo/rules/maintenance/task-planning.md`
- 项目接入方式：`.oo/docs/` 公开文档内容源或对应模块 README
- Relay 托管服务 / 私有化部署 / Vercel / Cloudflare / 域名与账号边界：`.oo/rules/RELAY-DEPLOYMENT.md`，再读 `apps/relay-server/AGENTS.md`、`apps/relay-admin/AGENTS.md` 和 `packages/plugins/relay/AGENTS.md`
- 使用文档边界约定：`.oo/rules/USAGE.md`
- hooks 方案与维护：`.oo/rules/HOOKS.md`、`.oo/rules/HOOKS-REFERENCE.md`
- benchmark 方案与规划：`.oo/rules/BENCHMARK.md`、`.oo/rules/BENCHMARK-PLAN.md`
- 当前重构待办：`.oo/rules/REFACTOR-TODO.md`
- 发布与更新日志：`changelog/`

前端任务补充：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oneworks-ai/app](https://github.com/oneworks-ai/app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
