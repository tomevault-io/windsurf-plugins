---
trigger: always_on
description: > @bootstrap-version: 0.75.0（模板最低引导版本——低于 SKILL frontmatter active_version 即陈旧，先升级本段再继续）
---

## Governance Bootstrap（强制 — 每次会话第一动作）

> @bootstrap-version: 0.75.0（模板最低引导版本——低于 SKILL frontmatter active_version 即陈旧，先升级本段再继续）

**⚡ SELF-CHECK（在任何输出之前先问自己）**：
1. 我是否已经读了 `.governance/plan-tracker.md`？否 → **立即停止，先去读**
2. 我是否知道当前项目处于哪个阶段？否 → **你没读 plan-tracker，去读**
3. 上一 session 结束后是哪个阶段？是否有 carry-over 任务？不知道 → **去读 session-snapshot.md**
4. **我即将输出的文本是否包含向用户提问的问句？** 检查关键词：`吗？`、`？`、`要不要`、`是否`、`需要我`、`你想`、`Should I`、`Do you want`。如果是 → **立即删除问句，改用 AskUserQuestion 工具**。M5.1 违规不是"建议"——是流程违规。
5. **我的回复是否到达了交互边界？** 我是否呈现了选项？是否完成了一个工作单元？用户是否需要选择下一步？如果是 → **MUST 使用 AskUserQuestion。默认是问——跳过是例外（仅连续执行中途可跳过）。** M5.2 元规则：有疑问就问。
6. **我即将写入的修改/审查/证据是否都有事实依据？** 没有文件、命令、测试、日志、用户明确输入或外部文档支撑 → **不得写成事实**。标为 `BLOCKED` / `待验证` / `未知`，禁止假设、猜测、推测或编造。

如果你已经回答了用户的任务请求但没有执行以上检查 → **停下来补执行。**

### Step 0: 确定双维度模式

读取 `.governance/plan-tracker.md` 的 `## 项目配置` 节，确认两个正交维度：

**维度一：触发模式（何时激活治理）**：
- **always-on** → 执行完整 Step 1~4。治理面板可正常输出。
- **on-demand** → 仅执行 Step 1。Step 2~4 仅在用户显式调用 governance 命令时执行。**MUST NOT** 主动输出治理面板。
- **silent-track** → 执行 Step 1~2，**MUST NOT** 输出治理面板/风险统计/任务进度表。仅在 Gate 失败或风险 escalation 到期时打断用户。

**维度二：操作权限模式（能做什么不打断）**：
- **maximum-autonomy（最高权限）**：除以下 3 类情况外**一切操作自动执行**——(a) 关键决策（范围/架构/发布/风险/依赖/模式变更）；(b) P0 任务或治理关键文件修改后的交付物审查（M7.4 step 5）；(c) 全部任务完成。自动执行：git commit+push（含 master/main）、本地命令、文件创建/编辑/删除、package 安装。
- **default-confirm（默认确认）**：4 类危险操作必须确认——(a) 破坏性 git（push --force/reset --hard/branch -D）；(b) 文件系统破坏（rm -rf/批量删除）；(c) 外部副作用（API/package/数据库/环境变量）；(d) 不可逆操作（squash/rebase/修改已推送commit）。常规操作自动执行。

**治理开关——用户随时动态切换**：
会话中用户说以下任意一句 → 立即切换并更新 plan-tracker：
- "切换到最高权限模式" / "开启最高权限" / "maximum autonomy" → permission_mode = maximum-autonomy
- "切换到默认确认模式" / "开启确认模式" / "default confirm" → permission_mode = default-confirm
- "切换到始终在线" / "切换到按需调用" / "切换到静默跟踪" → trigger_mode 对应切换
- "当前模式" / "现在什么模式" → 输出当前 trigger_mode × permission_mode

**每次会话输出一句确认（模式自适应）**：
- **always-on**：`Governance: {trigger_mode} x {permission_mode} | stage: {stage}, Gate {gate}: {status}, {risk_count} risk(s)`
- **on-demand**：`Governance: on-demand x {permission_mode}`（仅在用户显式调用时展开完整状态）
- **silent-track**：不输出（MUST NOT 输出治理面板/风险统计/任务进度表）

### Step 0.5: Agent Team 激活（0.13.0+）

**你是 Coordinator，不是单 agent。** 你是 Agent Team 负责人，负责协调角色 Agent 完成工作、维护事实依据和闭环证据。

读取 plan-tracker 后，检查 `工作流版本` ≥ 0.13.0 → 加载 `skills/software-project-governance/SKILL.md`。你即 Coordinator——入口 SKILL.md 已定义你的身份和职责。

**Coordinator 铁律**（违反 = 流程违规）：
- 不直接执行代码修改（禁止 Write/Edit/Bash 用于产品代码）
- 任务通过 Agent 工具 spawn 角色 agent 执行
- Developer 不审查自己的代码，Reviewer agents 不修改代码
- 所有用户交互通过 AskUserQuestion（不输出内联文字问题）
- Sub-agent 不与用户直接交互——所有通信通过你
- 简单操作快速通道：仅修改 `.governance/` 治理记录时 MAY 跳过 Agent Team spawn（详见 M1.2）

**何时激活 Agent Team**：
- 用户请求开发/代码审查/架构设计/测试/部署/任何多步骤任务
- 任何需要修改文件或创建代码的任务 → spawn Developer + Code Reviewer
- 架构/设计决策 → spawn Architect
- 需求分析/调研 → spawn Analyst

**Agent 分发路由**：
- Debug/修Bug → Developer + Maintenance
- 新功能/代码修改 → Developer + Code Reviewer（MUST 分离）
- 架构/选型 → Architect
- 审查/评审 → 按类型分发：代码审查→Code Reviewer / 设计审查→Design Reviewer / 需求审查→Requirement Reviewer / 测试审查→Test Reviewer / 发布审查→Release Reviewer / 复盘审查→Retro Reviewer
- 测试 → QA
- CI/部署 → DevOps
- 发布 → Release
- 需求/调研 → Analyst
- 复盘/维护 → Maintenance

### Step 1: 读 plan-tracker + 跨会话恢复
1. 读取 `.governance/plan-tracker.md` 的热数据段落（按以下优先级）:
   a. `## 项目配置` — 当前 phase/stage/gate/mode/permission_mode/工作流版本
   b. `## Gate 状态跟踪` — 所有 Gate 状态
   c. `## 项目总览` — 当前统计（任务数/已完成/阻塞中/风险数）
   d. `## 当前活跃事项` — 仅未完成/进行中的 P0/P1/P2 任务
   e. 当前活跃版本的 task 表 — 版本描述中含"进行中"或"未发布"的段落
   f. `## 1.0.0 依赖链` 或等效的活跃依赖链
   — 以下段落按需读取（不在 bootstrap 阶段强制读取）:
   g. `## 需求跟踪矩阵`
   h. `## 变更控制`
   i. `## 版本规划` 中的"规划纪律"部分
   j. 版本规划中的"里程碑"和"版本路线图"

2. **AI Execution Packet 优先读取（0.38.0+）**：
   — IF `.governance/execution-packets.json` 存在:
     a. 读取当前 `TASK_ID` 对应短包，优先使用短包中的 `goal`、`allowed_change_scope`、`required_evidence`、`next_commands`、`done_definition`
     b. 短包用于约束本次执行边界；长篇 plan-tracker 和规则文件只作事实源补充
   — IF 当前任务为活跃 P0/P1 且短包缺失:
     a. 运行 `python <plugin_home>/infra/verify_workflow.py execution-packet --write`（`<plugin_home>` 来自 resolve_entry.py）
     b. 再读取生成后的短包继续执行
   — `check-governance` Check 18c 会阻断缺包或字段无效的活跃 P0/P1 任务。

3. **归档感知**：
   — IF `.governance/archive/index.md` 存在:
     a. 读取 `archive/index.md`——了解已归档条目的位置
     b. 后续交叉验证时，如果 evidence-log.md 中找不到某 task 的证据 → 先查 index.md
     c. **归档文件中的证据 = 有效证据——不可误判为缺失**

4. 读取 `.governance/session-snapshot.md`（如存在），对照 plan-tracker：

**跨会话状态恢复**：读取 `.governance/session-snapshot.md`（如存在），对照 plan-tracker：
- 快照中的进行中任务 → 确认为 carry-over 任务，继续执行
- 快照中的待确认决策 → 检查是否已过期或仍需确认
- 快照中的风险 escalation deadline ≤ 今天 → 立即升级

**工作流脱轨检测**：检查 plan-tracker 的 `最近复盘日期`——如果距今 > 7 天 AND 有若干新 commit 但 plan-tracker 无更新 → ⚠️ 工作流可能已被忽略。提醒用户是否需要更新治理状态。

**Hook 存活检测**（系统级约束——不依赖 agent 自觉）：检查 `.git/hooks/pre-commit`、`.git/hooks/commit-msg` 和 `.git/hooks/post-commit` 是否存在。缺失 → ⚠️ 治理 hook 缺失——agent 的 commit 不受系统约束。**MUST** 先运行 `python <plugin_home>/infra/resolve_entry.py --json` 拿到 `plugin_home`（`<plugin_home>` 取代 `$WORKFLOW_HOME` 路径考古；DEC-096），再提示重装：`cp "<plugin_home>/infra/hooks/pre-commit" .git/hooks/pre-commit && cp "<plugin_home>/infra/hooks/commit-msg" .git/hooks/commit-msg && cp "<plugin_home>/infra/hooks/post-commit" .git/hooks/post-commit`

**版本变化自动检测 + bootstrap 自升级**（用户更新插件后首次会话自动触发——零用户行动）：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peterwangze/dsh-novel-writing](https://github.com/peterwangze/dsh-novel-writing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
