---
trigger: always_on
description: > 本仓库实现导引的**权威文件 = `AGENTS.md`**;`CLAUDE.md` 是指向它的软链(symlink)。
---

# AGENTS.md — ccteam 实现导引

> 本仓库实现导引的**权威文件 = `AGENTS.md`**;`CLAUDE.md` 是指向它的软链(symlink)。
> 面向**下一次接手 ccteam 实现的 agent session**:Claude Code 读 `CLAUDE.md`(→ 本文)、Codex 读 `AGENTS.md`(本文)—— 文中 Claude Code 特定机制,Codex 自行映射到等价工作流。每次起手必读。
> 历史里程碑 + 升级 migration 见 `docs-local/versions/v0-X-Y/README.md`(**gitignored 本机文档区** —— 版本归档 + 研究笔记已从 `docs/` 迁出、不入库),本文描述**当前状态 + 红线 + 纪律**。
> **治理脊柱 = `.loop/`**(state 焦点 / backlog 队列 / history 蒸馏史 / verify 门禁地图 + 写权守卫);**冷启动三读 = 本文 → `.loop/state.md` → `.loop/backlog.md`**,代码按卡面坐标按需读。

---

## 〇、当前架构总览

本仓已落地 **「IM 通用模式 + session 独立一等实体 + 插件市场 + 协议轴 + (v0.9.0) Agent2Agent 委派底座」**。**v0.9.0 核心 = A2A 基础能力**:任意 session 经 8 个 MCP 工具 spawn/dispatch/collect 任意其他 session —— **harness(claude/codex/grok/opencode/kimi)是 session 属性;host(local/卫星)自 v0.9.2 起是 project 属性,session 继承其 project 的绑定**,委派语义对两轴一致;引擎**零内置 persona**(废除 cto),编排智能全在用户空间;铁律「只做单 harness 做不到的,永不做厂商能力」。Claude 两条 spawn 路径:**`ClaudeStreamJsonAdapter`(`stream-json`,默认主路 —— 长驻子进程 + 双向 NDJSON,无 PTY/pane/hook)** 与 `ClaudeTuiAdapter`(`terminal`,tmux + 逐字节镜像);session facet `protocol`=`stream-json`(默认)| `terminal`(+ slash bridge / HITL / 故障矩阵)。架构 SoT = `docs/dev/tech-design.md` + 本文(**协议细节一律以代码为准**,见 tech-design 末尾「协议→代码位置」指针表)。本节 = **架构总览**(**不可触碰的红线清单见 §三**,勿与本节混)。**下文若仍见 orchestrator / 多模式(模式 1/2/3)/ flex / session=role / agent-team init,以本节为准 —— 已退役:**

- **核心模型 `chat ⇄ project ⇄ session`,role 是 session 的属性**:一个 chat = 你的终端(IM chat 或 web)→ 切 project → spawn/resume **session**。**session 是独立一等实体**,有持久 `sid`(`s<N>`,单调、扛 daemon 重启、不复用);**role 降为 session 的一个属性**(spawn 时绑 `--agent <role>` persona)。**同一 role 可并存多个 session**(去掉了 `(project,role)` dedup)。session 启动:**默认 `stream-json`**(长驻子进程,无 hook/pane);`terminal` 协议才走 `claude [--agent <role>] --name|--resume`(tmux send-keys + hooks,`Stop`→`chat_turn_completed`)。role 库 = 项目级 `.claude/agents/<role>.md`;**v0.9.0 起 `ccteam init` 不种任何 role**(废除 cto 内置工作流),默认会话 roleless(裸 vendor 读项目 `CLAUDE.md`);编排 persona 从 ccteam-hub 装(`fable-advisor`/`team-brain` 等示例配方)或用户自建。
- **turns / marker 全按 sid**:turns = `.ccteam/chat/<sid>/turns.jsonl`、transcript cursor / active-session marker 全按 sid;gateway `spawn_event_pump` 的 ANSWER 分支按 sid `append_turn`(live daemon 唯一 turns writer)。**terminal 协议**额外:pane = `ccteam-chat-<slug>-<sid>` + `CCTEAM_CHAT_SID` pane env(daemon HTTP 加 `X-Ccteam-Sid` → hook/in-pane forwarder 报 sid);**stream-json 默认路无 pane/hook**,sid 在 adapter 内。
- **roleless session 合法**:空 role → spawn **不加** `--agent`(裸 claude 自读项目 `CLAUDE.md`/`AGENTS.md` 当 brain);非注入(ccteam 本就不注入 system prompt,省略 `--agent` 不违反 No-prompt-injection)。
- **No prompt injection 由 `--agent` 兑现**:role 行为住 `.claude/agents/<role>.md`,vendor 原生 `--agent` 让它**自读**,ccteam **不**注入 system prompt —— 这条红线现在是**被满足**,不是被违反。
- **daemon = IM gateway + web + MCP Unix socket**(一个进程,**不 tick、无 orchestrator 循环**);会话 = resume-by-session-id(spawn-on-demand + 容量挤停(live ≤ 50,LRU 优雅停、可 resume)+ 按 sid resume,**非**常驻吊着;chat 复用 context 是 feature)。`ccteam-flow` orchestrator **存在但 daemon 不跑**(deferred)。
- **harness × provider × protocol facet**:`harness` = agentic CLI adapter(本版 claude-code 跑通;codex best-effort;gemini-cli/grok-cli/其余 = future,**可扩展 `AgentVendor` enum**);`provider` = 子 facet(model);**`protocol` = Claude 第三轴(v0.8.11)= `stream-json`(默认)| `terminal`**:`stream-json` = 长驻 `claude` 子进程 + 双向 NDJSON 管道(`ClaudeStreamJsonAdapter`,无 PTY/pane/hook 链,chat-only 主路);`terminal` = tmux PTY + TUI(`ClaudeTuiAdapter`,要逐字节终端镜像/attach/screenshot 时选)。`default_adapter_factory` 按 `(vendor, protocol)` 三路由(`crates/ccteam-im/src/daemon.rs`),两个 Claude adapter emit **同一 `CanonicalEvent`** → gateway `spawn_event_pump` 零改动消费。**`terminal` 协议(tmux/rmux/PTY)已进维护期,规划淘汰:不再为新 vendor / 新功能扩 terminal 路径**(仅维持既有 Claude terminal 会话);新 harness 一律长驻 stdio 协议(stream-json / ACP / app-server)。命名 `protocol` 非 `backend`(`backend` 留给 host 轴);session meta 的 `host` 字段自 v0.9.2 起是**继承值**(spawn 时从 project 绑定解析,记录用)。harness/provider/protocol 是 session 属性、非顶层资源;`GET /capabilities` 按 PATH probe 动态列当前可用 harness(×provider)。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firstintent/ccteam](https://github.com/firstintent/ccteam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
