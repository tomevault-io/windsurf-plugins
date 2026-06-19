---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这是什么

Commander ⚡ —— AI 多实例实时指挥台,「像皇帝批阅奏章一样」管理多个 Claude Code 会话。机器上正在跑的 Claude Code 会话会自动汇聚成卡片,按「谁在等你」排序供逐条审阅/续话/调度。

## 命令

```bash
pnpm install                          # 用 pnpm，不是 npm

./start.sh                            # 首选：一键拉起（缺 dist 自动构建 + 装 hook + 起服务 + 开浏览器）
./start.sh --port 4000 --no-open      # 开关：--port/--build/--install-hooks/--open 及 --no-* 反向
./start.sh --help                     # 全部开关

pnpm build                            # vite 构建前端到 dist/（server 托管它）
pnpm dev                              # server(3890) + vite(5173) 并行，前端热更
pnpm dev:client                       # 只起 vite 开发服务器

# 也可绕过脚本直接调（start.sh 内部就是调它们）：
node bin/commander.js serve --port 3890   # 起服务（默认端口 3890）
node bin/commander.js install-hooks       # 装全局 Claude Code hook（追加，不覆盖）
node bin/commander.js status              # 终端速查队列
```

无 lint 配置。测试用 Node 内置 runner：`pnpm test`（= `node --test "test/**/*.test.mjs"`，测试文件后缀为 `.test.mjs`）。跑单个测试文件：`node --test test/scheduler.test.mjs`。验证靠：`pnpm test` 通过 + `pnpm build` 通过 + 浏览器实测 + `node -e` 直接 import server 模块跑函数。

### ⚠️ 改后端必须重启 node 进程

server 模块在进程启动时被缓存。**改 `src/server/*.js` 后只 `pnpm build` 不生效**,必须重启:

```bash
pkill -f "commander.js serve"; node bin/commander.js serve --port 3890
```

只改前端则 `pnpm build`（或 `pnpm dev` 热更）即可,无需重启 server。

> ⚠️ **并行开发时别用 `pkill -f`**——它会无差别杀掉所有 worktree 的 server。在 worktree 里改后端,用 `scripts/wt.sh restart`(只按本 worktree 的专属端口定位并重起)。见下「多特性并行开发」。

## 架构大图

> 📌 **文档同步是硬规则**:当你改动了架构、数据流、状态语义,或纠正了本文档里某个不准确的认知(比如「Notification 其实要按 matcher 精筛」这类),**必须在同一次改动里把本文件 / 对应 spec / 相关代码注释一起改齐**,别让文档落后于代码。下游每个 session 都靠这份文档建立认知,文档错一句,后面所有人跟着错。改完问自己:「按这份文档重建认知,会不会被误导?」

两层。**调度内核稳定,内容来源/渲染可插拔**——这条边界是设计基线,见 `specs/000-architecture.md`,新接入来源不得改内核。

### 数据怎么进来（两条源,scanner 兜底 hook）

```
Claude Code hook ──► ~/.commander/events.jsonl ──► events.js（tail 读新行）─┐
                                                                          ├─► upsertFromAgent()
~/.claude/projects/*.jsonl ──► scanner.js（周期扫描，末状态判定）──────────┘   (tasks.js)
```

- **events.js**：消费 hook 写的事件流(精确,waiting/completed ~95%)。默认只读启动后的新行。
- **scanner.js**：扫 `~/.claude/projects` 的会话 jsonl,靠 mtime 静默阈值(180s)+ 末条事件角色判定 waiting/idle(兜底,~70%,漏报为主)。
- **`upsertFromAgent`**（`tasks.js`）是两条源的汇合点：会话 upsert → 自动建/更新「隐式 task」→ 入队。**hook 数据优先级高于 scan**(`LIVE_RANK`),scan 不能把 hook 设的精确态覆盖回近似态。

### 任务 ↔ 会话模型（`tasks.js` + `scheduler.js`）

- 一个 **task** 聚合若干 **session**。会话发现时自动建 `implicit: true` 的隐式 task。
- **排序优先级**（`scheduler.js` `rank()`）：P0 置顶 → **liveState 权重 `waiting > completed > running > idle`**(让等你的先冒出来) → 优先级 P0-P3 → skipCount 降权 → queuedAt 升序。
- 操作:`done`/`skip`(重排到同档末尾)/`defer`(定时 `tickDefer` 到点复活)/`dismiss`(标记会话 `dismissed`,不再被扫描复活,除非来新的 waiting hook)。
- 任何改队列的操作走 `notifyChange()` → 持久化 + ws 广播 `queue_updated`,current 变了再推 `new_current`。

### transcript 渲染（结构化 parts）

- **`transcript.js` `getSessionContext`**：把会话 jsonl 解析成结构化消息,每条 `{seq, role, ts, parts[], text}`。`parts` 的 `kind ∈ text|thinking|tool_use|tool_result|todos`。
- 后端做 **`tool_use.id ↔ tool_result.tool_use_id` 配对**,结果挂在 `tool_use.result` 上,并吸收/丢弃独立的 tool_result 噪音消息。
- 保留顶层 `text`（parts 拼成）作兼容字段,供 LLM 分析 / firstMessage。
- 前端 **`parts.jsx`**：`MessagePart` 按 `kind` 分发到 DiffPart(Edit/Write,jsdiff 行级 diff)/BashPart(命令高亮+折叠输出)/FilePart(Read/Grep/Glob/LS 折叠)/ThinkingPart/TodoPart/GenericToolPart。highlight.js 按需注册 ~13 种语言。
- **`SourceView`**（`TaskCard.jsx`）按 `source.type` 分发：`claude`→结构化渲染；`codex`/`web`→占位(架构预留,见 specs 002/003)。

### 网页续话（`converse.js`）

面板里给某会话发消息 → 在该会话 `workingDir` 下起一个**长驻** `<launcher> --resume <sid> --input-format stream-json --output-format stream-json` 进程(spec 015,取代旧的 `-p` 短命模式),把消息编码成 NDJSON `{type:'user',...}` 写进它 stdin;解析 stream-json 增量经 ws `type:'converse'` 推前端。进程随会话存活(多轮共享上下文)、空闲超时回收。**launcher 从 `cmdTemplate` 派生**(取 `--resume` 之前的前缀):新装默认原生 `claude --dangerously-skip-permissions`,老配置(缺 `cmdTemplate`)沿用旧默认 `ccr code`(`config.js` `LEGACY_CMD_TEMPLATE`)。

**交互式权限审批 / 澄清 / 计划(L1+L2)**:放行与否**派生自 `cmdTemplate`**——含 `--dangerously-skip-permissions` → 全放行,不挂权限工具(实测 skip 模式下 `--permission-prompt-tool` 根本不被调用);**不含 skip** → 挂内置 perm MCP server(`perm-server.js`,独立 stdio 子进程)+ `--permission-mode default --permission-prompt-tool mcp__commander__approve`。Claude 想用工具/反问(`AskUserQuestion`/`ExitPlanMode`)时调 perm 工具 → perm-server 经回环 HTTP(`/internal/permission`,token 校验,绑 127.0.0.1)转交主进程 → `perm-registry.js` 按 `tool_use_id` 挂起 + ws 广播 `permission_request` → 前端弹审批/澄清/计划卡片 → 用户答复走 `POST /api/sessions/:sid/permission` 回灌 → resolve → perm 工具返回 `{behavior,updatedInput?,message?}`。**fail-closed**:缺 tool_use_id / 超时(5min)/ 会话回收一律 deny,绝不静默放行。决定校验是 `permission.js` 的纯函数(可测)。

**进程注入保护(长驻下重定义)**:`running` 态会话,若**我们自己没有持有它的长驻进程**(说明可能是真终端在跑)→ 禁止注入;我们持有的长驻进程不算「别处」,允许继续。并发单飞,5 分钟空闲回收。

> **L3(同步「上朝面奏」:逐字流 + 守着等回复 + 连续即时往返)未做**,但本特性已把长驻进程模型/stdin 回灌/权限通道这些地基铺好,L3 是其上的增量,不返工。见 spec 015 末尾。

### 前端

React + Vite,`App.jsx` 经 `api.js`(含 ws `onConverse`)连后端。`TaskCard` 展示 current 任务 + 关联会话面板 + transcript;`Queue`/`Overview`/`Settings`/`AddTask` 为侧栏视图。

## 状态与配置的存放（关键约束）

- **运行时状态** → 仓库内 `data/{tasks,sessions,history}.json`（`store.js`,原子写 tmp+rename）。**`data/` 已 gitignore**(含本机路径与会话内容,不入库)。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yinshucheng/ccommander](https://github.com/yinshucheng/ccommander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
