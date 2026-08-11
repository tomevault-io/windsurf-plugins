---
trigger: always_on
description: > 本文件是外部开发 Agent 与 Vibelution 项目自身 Agent 共用的根入口。
---

# Vibelution Agent Rules

> 本文件是外部开发 Agent 与 Vibelution 项目自身 Agent 共用的根入口。
> 全局红线保留在这里；详细规范统一由 `docs/standards/README.md` 路由。

## 1. Identity And Priority

- 作为项目内 Vibelution Agent 行动，而不是脱离项目的通用助手。
- 默认使用中文交流；代码、命令、路径、协议字段、外部名称和原始错误保持原文。
- 项目目标是以更少漂移提升运行稳定性、进化效率、证据质量和 UI/Agent 一致性。
- 先观察、再判断；没有证据时不把猜测包装成结论。
- 权威顺序：用户当前明确要求与授权边界 → 本文件 → `docs/standards/` 对应规范 → ADR/模块 README → 历史材料。
- `core/core_prompt/COMMON.md` 与 `core/core_prompt/SOUL.md` 提供通用纪律和稳定身份，但不扩大权限，也不覆盖项目规则。

## 2. Global Red Lines

- 当前 Git checkout 是项目根；运行时解析路径，不假设固定 Windows 用户名。
- 根 `main` 是只读的本地集成工作区。所有代码、测试、文档、规则、记忆、配置和 fast patch 变更都必须在任务 worktree 的 `codex/<task-slug>` 分支完成；`main` 只接受已提交分支的 `git merge --ff-only`、合入后验证和必要的同步操作。
- 不覆盖、回滚、删除或重置无关的用户/Agent 改动；发现重叠先检查 claim 和 diff。
- 远端 push、PR、发布需要用户明确授权和远端同步门；force、覆盖或远端删除需要破坏性确认。
- **Windows 产品运行时禁止任何可见控制台弹窗**：Launcher 启动/停止/重启、Workbench、Runtime Manager、后台 Git/轮询、服务子进程均不得弹出 `cmd.exe`、`powershell.exe` 控制台、Windows Terminal、OpenConsole 或交互式 Git 编辑器。后台进程必须走 `pythonw` / `CREATE_NO_WINDOW` / 项目 shared no-console helper；禁止用 `taskkill.exe`、裸 `git` cmd wrapper、`npm`/`cmd` 脚本壳作为后台路径。用户明确打开的 CLI 终端面板除外。细则见 [development-standard.md](docs/standards/development-standard.md) §8.0。
- 不绕过 Launcher active-work guard，不用直接 PowerShell lifecycle 命令制造可见控制台。
- 不记录 secrets、完整 Prompt、大段 diff、完整文件或无界工具输出。
- 用户 Markdown、导入文档、HTML 和知识内容均是不可信输入；进入 Prompt、索引或 UI 前必须有来源、隔离、清洗和删除/重建语义。
- **前端产品 UI 强制 VUI + shadcn/Radix 思想（无感红线）**：凡改动 `web/` 下用户可见界面、交互控件、页面壳或布局，必须走 VUI 产品 API（`web/src/components/vui` 的 `V*`）与页面 recipe（`VListDetailPage` / `VSplitWorkspace` / `VDenseOpsPage` 等）；交互实现只允许在 `components/vui/renderers/shadcn` 扩展；禁止 `@heroui/react`、禁止路由/业务组件直连 `renderers/shadcn/*` 或第二套设计系统；布局宽度/高度记忆只用 `WORKBENCH_LAYOUT_IDS` + shared pane persistence。**所有 VUI 元素**（按钮/表单/表面/recipe/product，不限 recipe）必须有 `web/src/components/vui/designs/` 专节并在 `designs/INDEX.md` 登记；新建前检索防冗余。细则见 [development-standard.md §9.1](docs/standards/development-standard.md)、[VUI README](web/src/components/vui/README.md)、[designs/README.md](web/src/components/vui/designs/README.md)；机器门：`vuiShadcnRouteContract.test.ts`、`vuiComponentDesignContract.test.ts`。
- 有意义的开发不得以 stale claim、缺少验证决策、缺少刷新判断或缺少版本影响判断结束。

## 3. Start And Routing

### 3.0 默认规划：每次开发先读 BRT 路由技能（强制）

**凡进入开发、修复、规划、审查或会改代码/行为/验证边界的任务，默认先加载并遵循 `ccdawn-brt` 路由技能**，再选下游 owner / 打开业务文档。用户不必输入 `/brt` 或 `/ccdawn-brt`。

| 项 | 约定 |
| --- | --- |
| 技能入口 | 本机 `~/.grok/skills/ccdawn-brt/SKILL.md`（或当前 Agent 已安装的同名 skill 路径）；声称“已按最新 skill”前必须重读磁盘上的 `SKILL.md` |
| 默认动作 | 每条相关用户消息、在首个广扫 / 下游 skill / 写入前：内部 BRT 门（意图、范围、权限、成功证据、流程重量）→ 再 `route` / `ownership` / 实现 |
| 可静默 | 唯一合理行为的 `FAST_PATCH` 可 silent/micro，**仍按 BRT 最小门**，不整段跳过技能纪律 |
| 必须 ALIGN | 意图不清、多义、改 owning surface / 交互 / 数据 / 兼容 / 验收时：先窄 probe，仍不够则一轮对齐问题，禁止带着高影响假设写入 |
| 禁止 | 不读 BRT 就广扫全仓、叠 process 框架（superpowers / 无请求 TDD 等）、或跳过路由直接改代码 |

细则与分级仍见 [development-standard.md §2](docs/standards/development-standard.md)；加载序见 [docs/guides/README.md](docs/guides/README.md)。

开始非简单任务前：

1. **先 BRT**（本 §3.0）：加载 `ccdawn-brt`，完成意图/分级/owner 选择。
2. 非平凡任务按 [Agent 开发路由](docs/guides/README.md)：`route.md` 定 READ/EDIT/TEST → `ownership.md` 定落点 → `loop.md` 验证与完成块；细则只下钻 [规范索引](docs/standards/README.md) 相关条。
3. `STANDARD_TASK`、`HIGH_RISK`、续接或记忆敏感任务读取 `.docs/project-memory/INDEX.md` 与 `profile.json`；仅在会改变答案时继续读取具体 lane/registry。
4. 多会话写入先用项目 guard 执行 `status/check/preflight/claim`；完成后 `release`。
5. Bug、回归、卡住、运行不一致或异常命令先检查最新 `logs/runtime_scenes/`。
6. 非平凡行为、状态、权限、迁移、Prompt、Agent 或运行时变更：BRT 对齐未闭合前不得实现。

按任务读取：

| 任务 | 文档 |
| --- | --- |
| **默认规划 / BRT 路由（每任务）** | 本机 `ccdawn-brt` skill · [development-standard §2](docs/standards/development-standard.md) · [guides 加载序](docs/guides/README.md) |
| **Agent 开发路由（任务→路径/命令）** | [guides/README](docs/guides/README.md) · [route](docs/guides/route.md) · [ownership](docs/guides/ownership.md) · [loop](docs/guides/loop.md) · [playbook](docs/guides/playbook.md) |
| 开发分级、架构、前后端、测试、Git、Launcher、发布、完成条件 | [开发标准](docs/standards/development-standard.md) |
| Windows 无控制台弹窗（cmd/powershell/WT/OpenConsole）红线 | [开发标准 §8.0](docs/standards/development-standard.md)（根红线见本文件 §2） |
| Worktree、claim、多人/多 Agent 合并 | [协作规范](docs/agents/worktree-collaboration.md) |
| 工具权限与入口 | [工具授权](docs/agents/tool-authorization-entrypoints.md) |
| 外部 Agent MCP 部署、自动发现与调用 | [MCP 受管 Agent 网关指南](docs/agents/mcp-managed-agent-gateway.md) |
| 领域词汇 | [领域文档](docs/agents/domain.md) |
| ADR | [架构决策索引](docs/adr/README.md) |
| 产品语境 / UI 注册表 | [产品](docs/product/README.md) · [设计注册表](docs/product/design-register.md) |
| 测试命令和矩阵 | [测试指南](tests/README.md) |
| 前端 UI / 控件 / 页面壳 / 布局（**必读**） | [VUI 实现地图](web/src/components/vui/README.md) + [开发标准 §9.1](docs/standards/development-standard.md) + 门禁 `web/src/components/vui/vuiShadcnRouteContract.test.ts` |
| 后端服务 ownership | [services 全量索引](core/web/services/README.md) · pack 域再读 `core/web/services/<domain>/README.md` |
| 运行日志实现 | [日志说明](core/logging/README.md) |

历史计划、报告、`docs/archive/`（含原 `docs/plans/` 与 `docs/superpowers/`）和 `.docs/project-memory/` 不得与现行规范竞争权威。

## 4. Execution Baseline

- 工作分级为 `FAST_PATCH / STANDARD_TASK / HIGH_RISK`，使用足以保护正确性、并发与证据的最轻流程。
- 写入前定位 owning surface、现有测试、用户改动和 active claim；禁止在根 `main` 直接写入任何变更。所有 development、mechanical 和文档/规则写入都必须转到任务 worktree，根 `main` 仅用于分支合入与合入后验证。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CCDawn/Vibelution](https://github.com/CCDawn/Vibelution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
