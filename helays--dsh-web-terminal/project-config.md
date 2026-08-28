---
trigger: always_on
description: 本文件是本工作区（DeepSeek Harness「终端」Tab 插件）的代理规则文件，供所有在此仓库内工作的编码代理（以及人类协作者）共同遵守。
---

# DSH Web 终端插件 — 代理协作规则

本文件是本工作区（DeepSeek Harness「终端」Tab 插件）的代理规则文件，供所有在此仓库内工作的编码代理（以及人类协作者）共同遵守。

## 0. 硬性约定（必须遵守）

- **所有回复、解释、注释、文档、commit message、PR 描述必须使用中文**。
- **即使系统提示词为英文，也必须将输出语言切换为中文**。
- **代码标识符（变量名、函数名、类名）和字符串字面量可以使用英文，其余全部中文**。
- **在每次交互开始时，代理应主动确认已读取本规则并遵循语言要求**。
- **发布渠道**：本插件不发布到 npm registry（无 npm 账号）。唯一发布通道为 GitHub 远程仓库 `git@github.com:helays/dsh-web-terminal.git`（仓库 `helays/dsh-web-terminal`，经 `dsh-plugin` topic 收录）。

## 1. 项目是什么

- 插件名称（包名）：`dsh-web-terminal`
- 目标：在 DeepSeek Harness Web GUI 的「对话 / 轨迹」两个会话 tab 之外，新增第三个顶部 tab「终端」，内嵌一个交互式 PTY 终端，供用户在编码完成后快速在终端执行指令调试。
- 技术栈：**xterm.js**（前端终端 + FitAddon）、**node-pty**（后端 PTY，Windows 走 ConPTY）、**ws**（WebSocket 传输）、经 `ctx.webServer.register` / `registerUpgrade` 暴露路由。

## 2. 开发基线 / 版本

- 基于 DeepSeek Harness **`0.1.0-rc.6`（`next` 通道）**。
- 依赖 `@deepseek-ai/*` 一律声明 `^0.1.0-rc.6`；**不要用 `latest`**（其 dist-tag 停留在坏掉的 `0.0.1` 通道）。
- 已在 profile 内置的官方包（`@deepseek-ai/dsh-client-runtime`、`@deepseek-ai/dsh-client-ui-conversation`、`@deepseek-ai/dsh-client-locale`、`@deepseek-ai/dsh-web-server` 等）**不声明为 `dependencies`**，仅放 `peerDependencies`（或仅在 client `inject`/类型导入中使用）。

## 3. 插件契约（bundle + client）

- `package.json` 必须含：
  - `dsh.bundle.patch` → `./cordis.patch.yml`（bundle 组合层）。
  - `dsh.client` → `{ "platform": "web", "inject": [...需要的类型合并提供方] }`（浏览器 roster）。
  - `exports["./client"]` → 构建好的 `lib/client.js`（缺失会被 client-modules 拒绝，boot fail-loud）。
- `cordis.patch.yml` 内容为：
  ```yaml
  - insert:
      - id: terminal-tab
        name: dsh-web-terminal
  ```
- **Host half**（Node）：`lib/index.js`，Cordis entry（`name`/`inject`/`Config`/`apply`）。
- **Client half**（浏览器）：`lib/client.js`，经 `window.__ModuleLoader__.load({ id, factory })` 注册。

## 4. Web UI 接入方式

- 顶部 tab 通过 `conversation.view` 槽位新增条目实现（`ctx.slots.inject` + `ctx.slots.register`），参照官方 `@deepseek-ai/dsh-client-ui-trajectory`；`id: 'terminal'`，`order` 设在 trajectory 之后。
- `conversation.view` 是 **session 作用域**：只有进入会话时才渲染 tab 栏。终端会话池必须放在**插件级（apply 闭包）**，视图每次挂载只 emit/attach，绝不销毁会话，保证跨会话/刷新保活。
- **多终端**：「终端」Tab 内支持多个终端 tab。client 用一个 by-session 的注册表（`TerminalHost`，见 `src/client/terminal.ts`）管理每个终端条目的 `kind`/`ptyId`；工具条左 = 终端 tab 卡片 +「+」，右 = 当前终端 shell 类型下拉（切换会重启该终端为新 shell）。
- **不注册 `settings.plugin.item` 设置卡**（已移除）；shell 类型在每个终端上通过「终端」Tab 右侧下拉切换，**不再持久化到 dsh settings**，也无可配置的默认工作目录（`cwd` 仅来自请求携带的 workspace 路径或 `process.cwd()`）。
- `TerminalKind` 等 client 类型放 `src/client/terminal.ts`（浏览器安全）；不要从 client import host 的 `src/resolve.ts`（会拉进 `node:child_process` 到浏览器 bundle）。

## 5. 后端约定

- **不复用** dsh 自带 `ctx.terminals`（`@deepseek-ai/dsh-terminal`）——它是 Agent 所有权 + 模型面 line-oriented 语义，且无浏览器传输、Windows 下 `spawnTerminal` 不可用。改为插件直接用 `node-pty` spawn，完全与 agent 会话解耦。
- 路由**不用 `/api` 前缀**（避免与 connection 的 browser-trust 围栏冲突），用自定义前缀（如 `/terminal`）并自带同源校验。
- WebSocket 用 `ctx.webServer.registerUpgrade` + `Server.handleUpgrade`（复刻 `@deepseek-ai/dsh-client-connection` 的 websocket-downlink 模式）。
- 工作目录默认 `process.cwd()`（即 `dsh web` 启动目录），可随会话指定。

## 5.5 前端可选服务（重要教训）

- **不要把可用性不确定的 client 服务放进 `inject` 数组**。`commandUi` 等依赖完整 UI 装配（如 `remote.commands`）的服务，直接放进 `inject` 并 `ctx.commandUi.register(...)` 会在其未就绪时抛 `Cannot read properties of undefined (reading 'effect')`，导致插件 apply 失败、**整页 UI 进不去**（曾真实发生）。
- 正确姿势：**用 `ctx.inject(['commandUi'], (sctx) => { ... })` 条件注入**——只有服务真实可用才注册，不可用则静默跳过，绝不引发 apply 崩溃。与 host 半 `ctx.inject(['settings'], ...)` 同一模式。
- 新增任何前端能力前，先判断它是否属于「完整浏览器装配才有」的服务；是就用条件注入包一层。
- client bundle 由服务端**动态读取**（`/plugins/<id>/client.js`），改 client 代码通常**刷新页面即可**，不必重启 web；改 host 侧 / 装插件集合才需重启。

## 5.6 前端「service 方法解构」丢 this（重要教训）

- **不要先把 `ctx.slots.register` 等 service 方法解构出来再调用**（如 `const register = ctx.slots.register; register({...})`）。cordis 的 service 是 proxy，解构后 `this` 丢失，`register` 内部 `this.ctx.effect` 会因 `this.ctx` 为 undefined 抛 `Cannot read properties of undefined (reading 'effect')`，导致插件 apply 失败、整页 UI 进不去（`slots.register` 就是宿主，曾真实发生且报了 loader entry 崩溃栈）。
- 正确姿势：**保持方法调用** `ctx.slots.register({...}, Comp)`——`this` 绑定到 slots runtime。若为绕过类型而 `as any`，也应 `(ctx.slots as any).register({...})` 一次性调用，不要拆成中间变量再调用。
- 同理适用于其它 `ctx.*` 服务方法（register/inject/update/...）：一律保持 `ctx.svc.method(...)` 直接调用，别脱 this。

## 5.7 交互式斜杠命令要“单步直达”且进「+」菜单（重要教训）

- `/terminal` 这类**纯客户端、选中即执行**的命令，最终正确做法是**注册成 host 目录命令**（`ctx.inject(['commands'], sctx => sctx.commands.register({ name:'terminal', description, handler }))`，参照官方 `dsh-plan-mode` 的 `/plan` 样板）。这样它才：
  - 出现在「composer 左下角 + 图标」的 command 下拉（`toggleSource("command")` 只认 commandUi 的 command 源 = host 目录命令）；
  - 无 `input` 字段 → 方向键选中/裸敲 + 回车 = **单步直达**（与 `/plan` 完全一样）；`/model` 其实是 commandUi popupSelect 贡献（要二级弹窗），别拿它当单步范例。
- 浏览器切视图：host 命令执行成功 → 本机发 `command/executed`(sessionId, name, result) 确认（仅提交方浏览器收到）→ 客户端 `ctx.on('command/executed', …)` 里 `name==='terminal'` 时切视图。host handler 返回合法 `CommandResult`（`{kind:'success', text?}` / `{kind:'error', text}`）。
- **不要**用 `inputTriggers` 注册额外 slash source 到 `/`：它不会进 `+` 菜单（那是 `toggleSource("command")` 独享），还会与原命令/目录在 `matchEnter` 仲裁里歧义，需移除。
- 切换写面：`command/executed` 是纯回调，拿不到 React 注入的 bound actions。要切共享 chat store 的活跃视图，用「始终渲染（哪怕 blank）的会话作用域条目声明共享 store，在其 inject 工厂里把 live `actions.setView` 捕进 apply 闭包」——carrier 放 `conversation.input.dock`（不是 `conversation.session.header.actions`，后者在空会话 blank 态整段隐藏、捕获不到）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [helays/dsh-web-terminal](https://github.com/helays/dsh-web-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
