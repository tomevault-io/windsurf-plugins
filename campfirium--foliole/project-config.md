---
trigger: always_on
description: - 若规则与用户当次最新指令冲突，以用户最新指令为准；若与当前代码现状冲突，以可运行代码现状为准。
---

# AGENTS

- 若规则与用户当次最新指令冲突，以用户最新指令为准；若与当前代码现状冲突，以可运行代码现状为准。

## Project Baseline And Work Unit

- 当前仓库是多平台单仓：`Electron + React + TypeScript + Vite + Capacitor`；主要宿主与表面为 `electron/`、`android/`、`src/app/`、`src/companion/`、`src/shared/platform/`。
- 默认在 `dev` 主干按 Track-Based 连续小步推进；不创建 feature branch / worktree，除非用户明确要求。
- 单次只交付一个可运行、可验证、可回退的能力闭环；闭环以用户可验收行为、数据语义或迁移语义为边界，不以文件、函数、测试断言、提交数量或“超过 3 个文件”为边界。
- 能力闭环必须覆盖本轮承诺所需的入口、模型、消费侧、必要持久化、边界防护和验证；新增功能覆盖用户入口、状态模型、业务行为、失败或空状态，Bug 修复覆盖现象确认、根因修复、回归验证和用户可见结果恢复。
- 禁止混入无关重构，也禁止把同一闭环拆成微任务；同一闭环内必要的模型、入口、消费侧、测试、边界防护和生成物更新必须一起收口。
- 共享目标是“共享核心 + 薄宿主适配”；写产品代码时先面向稳定能力建模，平台层只做最必要 adapter，禁止为每个平台复制业务逻辑。
- 所有正式图标入口、菜单入口必须有对应命令，且图标 / 菜单 / 命令同源、命名一致；详见 `.lab/specs/_product/methodology.md`。

## AGENTS Routing

- 启动时先读根 `AGENTS.md`。
- 根 `AGENTS.md` 负责全仓硬规则与路由；平台与局部细则下沉到对应目录的 `AGENTS.md`。
- 只要任务触及下列路径，实施前必须按路由补读对应规则来源，并在该规则基础上执行：
- `electron/**`、`scripts/windows/**`、`playwright.desktop.config.ts`、`D:\X\U\Foliole\Data\foliole.db` 相关诊断或桌面运行链路：读取 `electron/AGENTS.md`
- `android/**`、`scripts/android/**`、`capacitor.config.ts`：读取 `android/AGENTS.md`
- `src/companion/**`：读取 `src/companion/AGENTS.md`
- `ios/**`：读取 `ios/AGENTS.md`
- `src/app/**`：当前没有单独局部 `AGENTS.md`，继续直接执行根 `AGENTS.md` 的 desktop renderer 规则；涉及运行时 UI 行为时，验证按 `electron/AGENTS.md` 的桌面验证规则执行
- `src/features/editor/**`：读取 `src/features/editor/AGENTS.md`
- `src/features/**`、`src/store/**`、`src/shared/**`：当前没有单独局部 `AGENTS.md`，继续直接执行根 `AGENTS.md` 的 shared / cross-host 规则
- 若一次任务同时跨多个宿主或表面，必须把相关局部 `AGENTS.md` 全部读齐；冲突时按“更靠近改动目录的规则优先，跨目录共享规则回退到根规则”执行。
- 关键平台约束必须落在根或对应目录 `AGENTS.md`，不得只放在普通项目文档里。

## Document Read Order

- 启动时只读 `AGENTS.md`。
- 任务涉及 renderer UI 改动（`src/app/**`、`src/companion/**`、`src/features/**`、`src/shared/ui/**`）时，实施前必须先读取 `DESIGN.md`，再读取 `.lab/specs/shared/ui/llm-ui-rules.md`。
- 任务涉及 UI 文案、产品对象命名、空状态、按钮、菜单、队列与阅读单元称呼时，实施前必须读取 `.lab/specs/_product/terminology-and-copy.md`。
- 任务涉及 `docs/i18n/guides/**` 的 Demo Guides 内容时，实施前必须读取 `docs/i18n/guides/README.md`；英文 `en` 文件是每个 slug 的必需源，其他语言可按需补齐并回退英文。
- 编写 Foliole Demo / Guides 文案时必须保留 Demo 边界：它可以是浏览器里的预置内容体验，但不得暗示为 Foliole Web 版、正式数据环境、桌面版替代品、完整本地文件 / 导入能力、完整桌面功能集或可长期生产使用的在线工作区。
- 新增或修改用户可见 UI 文案时，必须按 `.lab/specs/_product/terminology-and-copy.md` 的文案分层先区分用户效果、轻原理说明与内部语言；最终文案不得直接从变量名、数据库字段、IPC / action 名、队列流程动词或对话里的临时术语生成。
- 任务涉及具体现有规范时，按需读取对应 `.lab/specs/**` 条目，不全量通读。
- 任务涉及新增、重写或审计 agent 规则时，按 `$agents-maintainer` 流程只审根与局部 `AGENTS.md`；不默认扫描其他项目文档。
- 仅在判断停车策略时读取 `.lab/internal/runtime/park.flag`；预览不再由持久 flag 自动触发。

## Agent Rule Maintenance

- 新增或修改 agent 规则前，必须先判断是否已被既有规则覆盖；能合并、压缩、下沉或删除旧规则时，不得直接追加。
- 根 `AGENTS.md` 只保留全仓硬规则、路由触发器和机械决策入口；平台细则归局部 `AGENTS.md`，长解释和背景归 specs / Atlas，不进入根规则。
- 机械可判定逻辑优先写成表格或脚本入口；禁止把多维状态机继续扩写成散文段落。

## Windows Command Boundary

- Windows 原生命令默认用已存在的 `npm` / `node` / 项目脚本入口执行；不得把多步验证长期写成内联 PowerShell / cmd 片段。
- 复杂 Windows 命令若涉及多层引号、环境变量、重定向、后台进程、native exe、`cmd.exe` / PowerShell 交叉调用或 stdout 可靠性判断，优先写成仓库内 Node runner 或已提交脚本；临时诊断必须把 stdout、stderr、exit code 写入 `.tmp/` 后再读取，不得只凭空 stdout 或空日志判定成功。
- 临时 Playwright / browser 验收、生产站点 browser probe、HTTP server + browser 脚本必须通过 `node scripts/with-resource-gate.mjs preview -- <command...>` 执行；Node REPL 只用于短探针，长流程必须转成仓库脚本。只清理 runner 自己启动的子进程树，不按进程名全机杀 `node.exe` / `msedge.exe`。
- 需要临时调用 Windows PowerShell 承载复杂参数时，使用 `powershell.exe -NoProfile -EncodedCommand` 并记录可复验日志；避免使用多层 `powershell.exe -Command "..."`、复杂 `cmd.exe /c ... && ...` 或嵌套 shell quoting。

## Task Execution And Risk Routing

1. 任务开工判断是首个动作规则：在第一次读文件、跑命令或改代码前，把最新用户请求归类为 `DIRECT`、`FOLLOW_PLAN`、`NEEDS_EVAL` 或 `STOP_CONFIRM`；只有 `DIRECT` 可以静默执行。
2. `FOLLOW_PLAN` 必须先读实施说明并遵守其中评估；`NEEDS_EVAL` 必须先输出轻量任务评估；`STOP_CONFIRM` 必须先等待用户确认。
3. 轻量任务评估必须写出 5 项：`任务类型`、`影响范围`、`已定路线`、`拒绝路线`、`停工点`；细则见 `.lab/specs/_governance/task-evaluation-expectation.md`。
4. 用户只给短标题、笼统“继续 / 修一下 / 处理一下”时，按顺序解析：本轮点名文件 / 实施说明 > 本轮 active file 若是实施说明 > 最近一条未完成的用户明确指令；仍无单一任务时只问一个澄清问题。
5. 需求、边界、验收标准或预期行为存在歧义时，先澄清；根因未确认时只写“现象 + 当前怀疑 + 待确认点”，禁止把猜测写成事实。
6. 命中高风险路径时，在开工判断或任务评估中明确“建议 High / XHigh”，但不得伪装工具层已切档；高风险包括 sync、数据库 / schema / migration、Electron preload / IPC / native bridge、Capacitor / Android lifecycle、review queue、delete / restore、import / reimport、持久化、重启后行为、冲突处理、安全边界、不可逆数据风险、人工验收失败后的复修和非显然 bug。
7. 若方案包含扫描、轮询、自造协议、新依赖、长期双写、运行时迁移、隐式 fallback、局部复制或先污染后清理，必须进入 `STOP_CONFIRM`，并说明方案身份是正式主方案、spike、诊断、兜底还是一次性迁移工具。
8. 执行中发现不再属于同一能力闭环，或新增 schema / bridge / IPC / 协议 / 依赖 / 后台任务 / fallback / migration / 双写 / scan / poll，必须停下进入 `NEEDS_EVAL` 或 `STOP_CONFIRM`。
9. 只有用户明确要求 spike，或任务评估把某段代码标为 spike / diagnostic / fallback / one-off migration 时，才允许写临时验证代码；否则临时代码不得接入正式入口。

## Pre-Edit Dirty File Gate

- 任何任务首次编辑文件前，必须先列出本轮计划编辑的文件清单；大任务以实施说明为准，小任务以编辑前的简短实现思路为准。
- 规则维护任务编辑任意 `AGENTS.md` 时不受本节 dirty-file gate 限制；仍必须只改本轮规则目标，不得借机重写无关规则。
- 对计划编辑文件执行 `node scripts/wait-clean-files.mjs <file...>`；脚本通过后才允许编辑这些文件。
- 若本轮只编辑 `package.json` 的 `scripts` 区，使用 `node scripts/wait-clean-files.mjs --allow-package-json-scripts-edit package.json`；该例外只免疫 `scripts` 以外的既有 package dirty，不免疫 `scripts` dirty、`package-lock.json` 或依赖 / 版本编辑。
- 若脚本发现计划编辑文件已有未提交改动，会持续等待，直到这些文件在当前 Git 工作区变干净；超时或被打断时，不得编辑仍 dirty 的文件，必须汇报仍占用的文件。
- 该 gate 只检查计划编辑文件，不扫描其他线程、不使用 worktree、不维护 ownership，也不要求在任务刚开始时精确判断改动范围。
- 编辑根 `AGENTS.md` 或局部 `AGENTS.md` 时免于执行本 gate；改前仍必须阅读当前 diff，并只做用户要求的规则改动，不能覆盖已有未提交改动。

## Delegation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [campfirium/foliole](https://github.com/campfirium/foliole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
