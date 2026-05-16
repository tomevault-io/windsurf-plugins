---
trigger: always_on
description: > **Note for human readers**: this file is the project constitution for AI coding agents working in this repository (Claude Code, Cursor, etc). It captures non-negotiable rules and the mental model assistants should adopt when contributing. Human contributors should also read [README.md](./README.md) and [docs/PRD.md](./docs/PRD.md).
---

# Galley

> **Note for human readers**: this file is the project constitution for AI coding agents working in this repository (Claude Code, Cursor, etc). It captures non-negotiable rules and the mental model assistants should adopt when contributing. Human contributors should also read [README.md](./README.md) and [docs/PRD.md](./docs/PRD.md).

> *Galley started as a workbench for GenericAgent. The first two letters of our name are a quiet bow to where we came from.*

**Brand wordmark rules**:
- **Body text / docs / sentences**: `Galley` (sentence case). Use everywhere the name appears inside prose, including README / CLAUDE.md / PRD / commit messages / comments.
- **Small wordmark display (≤ 20px)**: `GALLEY` (Newsreader serif, semibold, uppercase, tracking-[0.04em]). Currently used: Sidebar header (16px), Settings → About h2 (20px). Reads as a refined logotype mark with workbench weight.
- **Large hero display (≥ 30px)**: `Galley` (Newsreader medium, sentence case). Currently used: Onboarding StepWelcome h1 (36px). Uppercase at this scale reads as marketing banner; sentence case stays gentle and product-appropriate.
- Use **GenericAgent** / **GA** when referring to the upstream engine, never to mean Galley.

本地 agent team 编排器，人和 agent 都是一等公民。Galley GUI 给坐在桌前的 human operator，Galley CLI 给 Supervisor Agent 远程操作整个 session team。

v0.5 之前（v0.1 / v0.2）仍以"GA 的本地桌面工作台 + 多 session 并行"为主要使用形态；v0.5 起 dual-native 架构（Rust 端 Galley Core + 双前端）正式 ship。

- 产品定义（PRD v0.3）：[docs/PRD.md](./docs/PRD.md)
- 设计系统（DESIGN.md，draft）：[docs/DESIGN.md](./docs/DESIGN.md)
- IPC 契约：[docs/ipc-protocol.md](./docs/ipc-protocol.md)
- 发版 SOP（v0.2+）：[docs/release-workflow.md](./docs/release-workflow.md)
- Win 手动 build 指南：[docs/windows-build-checklist.md](./docs/windows-build-checklist.md)
- **Galley Core 重构执行手册** (v0.5 路径)：[docs/refactor/](./docs/refactor/README.md) — 跨多 session 重构的中央调度器；新 session 进入 B 阶段时必读
- 决策叙事 / 历史：[docs/devlog/](./docs/devlog/)

## 项目宪法（Non-invasive）

不能影响 GA 的独立运行。**违反任一条等于破坏项目核心承诺**：

- **不修改** `~/Documents/GenericAgent` 下任何文件（包括 `mykey.py`、`memory/`、`assets/`、源代码）
- **不覆盖** GA 的 venv / PATH / 环境变量
- **不 monkey-patch** `agent_runner_loop` 或 `do_*` 工具实现

**关于读取（read-only）**：

- **优先**走 GA 公开 API（`agent.list_llms()` / `agent.llmclient.backend.history` / `agent._turn_end_hooks` 扩展点）—— GA 自己保证 API 稳定，升级风险最低
- **直接读** GA 内部文件（`mykey.py` / `assets/` 下静态资源等）**只读**前提下允许，但需：
  - 在代码注释标注 coupling 点
  - 在 GA baseline 升级时审计该路径 / 格式是否仍有效
  - **任何"读取后基于读取结果回写 GA 文件"按"修改"对待，禁止**

宪法历史：2026-05-13 audit 时发现原条文"不读写 GA 的 mykey.py / memory/ / assets/" 字面禁止读取，跟实际行为（通过 `agent.list_llms()` 间接读 mykey.py 配置）不一致。重写为"禁修改 + 读取分级"，更准确反映 non-invasive 的真正含义：**保护 GA 独立运行 = 不改 GA 状态**，而读取本身从不破坏独立性。

允许的接入方式（详见 PRD 附录 A.2）：

- 启动 GA 子进程（每个 session 独立）
- 注册 `agent._turn_end_hooks`（GA 官方扩展点，主链路）
- 子类化 `GenericAgentHandler` 重写 `dispatch`（仅审批拦截，前置加门，不复刻原逻辑）
- 读 / 注入 `llmclient.backend.history`（用于历史恢复）

GA 升级时，Galley 只依赖 `BaseHandler` / `ToolClient` 这一层公开 API。

**例外条款：用户主动安装 Supervisor SOP**

v0.5 起 Galley Settings 提供 "Install Supervisor SOP" 按钮，用户点击后 Galley 把 `galley-supervisor-sop.md` 写入用户 GA 的 `~/Documents/GenericAgent/memory/`。这属于**用户显式触发的内容安装**，不属于"Galley 偷改 GA 状态"——宪法明确允许。

法理：宪法防的是"Galley 悄悄改 GA 让 GA 独立运行被污染"。SOP 是用户主动装的内容（类比"装 GA 插件"），不破坏 GA 独立性。删 Galley 后 SOP 文件留在 memory/，GA 仍然独立运行；用户也可以随时手动删该 SOP 文件，不影响 GA 任何功能。

实现要点：
- 装之前检查同名文件，存在时提示用户（不覆盖）
- 装的位置只能是 `memory/` 下且文件名固定为 `galley-supervisor-sop.md`（不允许 Galley 写到其它路径或其它文件名）
- 不接受用户配置"装到哪"——固定行为减少 surface
- 卸载 Galley 时不主动清这个 SOP（用户自己删，宪法非对称）

## Galley 架构原则 (v0.5+)

非侵入条款守 Galley ↔ GA 边界；本节守 Galley 自身的设计边界。**违反任一条等于破坏 v0.5 dual-native 承诺**：

### 1. Localhost only

**Galley Core 永远只 listen on AF_UNIX socket / named pipe，不开 TCP，不持有 token。**

远程访问（手机 IM 派任务给 Galley 这种场景）通过 Supervisor Agent 在外部传输层（GA 的 IM frontend / SSH / 其他）完成，**不是 Galley 的责任**：

```
手机 ─→ IM service ─→ 桌面上的 Supervisor (GA + IM frontend)
                              ↓ localhost (unix socket)
                              Galley Core
```

收益：
- 安全模型 = OS user filesystem permission，无 TLS / token / 证书 / 旋转
- 复用 GA 已经做好的 IM frontend，Galley 不重复造轮子
- "Galley 是本地的、数据不离开你的机器" brand 守住

**任何 PR 提"加 HTTP server / 加 token auth / 加远程访问"以本条款拒绝**。例外需先改宪法。

### 2. CLI surface 是公开契约面

Galley CLI 的 JSON 输出 schema 是 Galley 对 agent 生态的公开承诺。规范见 [docs/agent-api.md](docs/agent-api.md)（v0.5 ship 时 publish）：

- **schema_version 内 additive-only**：v1 schema 内只加字段，不删 / 不改语义
- **Breaking change 强制 bump**：要 break = `schema_version: 2`，旧版 SOP 可用 `?schema=1` 拿老格式
- **Exit code 分类稳定**：0/1/2/3/4/5 五类（详 agent-api.md），不重新分配
- **错误码 enum stable**：error 字段值是 stable identifier，不重新命名

下游 supervisor adapter（GA SOP / Claude Skill / 用户自写 agent）依赖这个契约稳定。**改 schema 比改 GUI 慎重得多**：GUI 改了用户重新学一遍，schema 改了所有 SOP 一起坏。

### 3. 数据不离开 Galley

**Galley 不存 Supervisor ↔ human 的对话内容**。supervisor 通过 CLI 发的命令、命令的 `--reason` 标注存进 Galley（per-session 行动日志），但 supervisor 跟 user 在 IM 里聊的对话不存。

收益：
- Galley 是 orchestrator 不是 chat platform，scope 守住
- 换 supervisor 不存在 "data migration" 问题

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangjc683/galley](https://github.com/wangjc683/galley) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
