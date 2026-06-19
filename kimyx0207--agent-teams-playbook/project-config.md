---
trigger: always_on
description: |
---


# Agent Teams 编排手册

作为 Agent Teams 协调器，你的职责包括：明确每个角色的职责边界、把控执行过程、对最终产品质量负责。

> **核心理解（铁律）**：Agent Teams 是"并行处理 + 结果汇总"模式，不是扩大单个 agent 的上下文窗口。每个 teammate 是独立的执行单元，拥有独立上下文，可以并行处理大量信息，但最终需要将结果汇总压缩后返回主会话。

## 跨平台兼容层（新增，不替代原流程）

本 Skill 的完整 6 阶段工作流、Skill 回退链、Agent → Skill 委派模式、质量把关和故障处理规则仍然保留。四个平台的差异只影响"用什么工具执行"，不影响"是否执行这些阶段"。

先识别当前平台，再把下列抽象动作映射到平台原生能力。不要在非 Claude Code 平台原样承诺 `Task`、`TeamCreate`、`SendMessage` 或 `Skill(...)` 一定存在；也不要因为平台工具名不同而跳过阶段0-5。

| 抽象动作 | Claude Code | Codex | OpenClaw | Cursor |
|------|------|------|------|------|
| 调用 Skill | `Skill(skill="name", args="...")` 或 slash skill | 读取并遵循本地 skill 指令；只有宿主暴露 skill 工具时才称为"调用" | 读取并遵循 `openclaw/skills` 或全局 skill；按 OpenClaw 当前工具执行 | 读取并遵循 `.cursor/skills` 或全局 skill；按 Cursor 当前 agent 能力执行 |
| 启动独立 Subagent | `Task(...)` | 宿主提供的 custom-agent / subagent dispatch；不可用则主线程分阶段执行 | workspace / agent 调度能力；不可用则主线程分阶段执行 | background agent / agent mode；不可用则主线程分阶段执行 |
| 组建 Agent Team | `TeamCreate` + `Task(team_name)` | 多个独立 subagent + 主线程汇总；无共享团队总线承诺 | team / workspace 能力存在时使用；否则多个任务或主线程 | background agents / team-like workflow 存在时使用；否则多个任务或主线程 |
| 成员通信/进度 | `SendMessage` 或 task result | 子任务结束汇报；有 agent I/O 工具才可中途交互 | 平台消息/日志；不可用时用阶段性文本汇报 | IDE/agent 日志；不可用时用阶段性文本汇报 |
| 规划文件 | `planning-with-files` skill | 若本地 skill/tool 存在则使用；否则使用内联计划或平台计划工具 | 若本地 planning skill 存在则使用；否则维护可见计划记录 | 若本地 planning skill 存在则使用；否则维护可见计划记录 |

**平台适配底线**：
1. 写计划时使用抽象动作名；执行时使用当前平台真实工具名。
2. 只有 Claude Code 可以承诺官方 `Task` / `TeamCreate` / `SendMessage` 语义。
3. Codex 中只有宿主实际暴露并调用了 subagent/custom-agent 工具，才代表真的触发后台 agent；没有真实工具调用就不能说 agent 群已启动。
4. OpenClaw/Cursor 的 team 能力可能来自项目插件、workspace 或 IDE 能力；先探测，再承诺。
5. 若平台不支持真正并行或团队通信，明确降级为"主线程分阶段执行"，但仍执行阶段0-5的治理流程。

## 适用 vs 不适用

| 适用 | 不适用 |
|------|--------|
| 跨文件重构、多维度审查 | 单文件小修改 |
| 大规模代码生成、并行处理 | 简单问答、线性顺序任务 |
| 需要多角色协作的复杂任务 | 单agent可完成的任务 |

**边界处理**：用户输入模糊时，先引导明确任务再决策；任务太简单时，主动建议使用单agent而非组建团队。

## 用户可见性铁律

1. 每个阶段启动前输出计划，完成后输出结果
2. 子agent在后台执行，但进度必须汇报给用户
3. 任务拆分计划必须经用户确认后再执行；若宿主平台或项目指令要求直接执行，则说明采用的默认假设
4. 失败时立即通知：`❌ [角色名] 失败: [原因]`，提供重试/跳过/终止选项
5. 全部完成后输出汇总报告（见阶段5格式），并说明真实使用的平台工具和任何降级路径

## 场景决策树

**执行顺序**：先执行阶段0和阶段1（强制），再根据任务复杂度选择场景（影响阶段2-5）。

| 问题 | 路径 |
|------|------|
| Q0: 阶段1找到完全匹配的Skill？ | 是 → 场景2 / 否 → Q1 |
| Q1: 任务复杂度？ | 简单(1-2步) → 场景1 / 中等(3-5步) → 场景3 / 复杂(6+步) → Q2 |
| Q2: 需要明确团队分工？ | 是 → 场景4 / 否 → 场景5 |

- 用户直接指定场景编号时,跳过决策树直接执行
- 未指定场景时，默认用**场景3（计划+评审）**
- **注意**：阶段0（planning-with-files）和阶段1（Skill搜索，包含 find-skills）是所有场景的强制前置步骤

## 5大编排场景

| # | 场景 | 适用条件 | 核心策略 |
|---|------|---------|---------|
| 1 | 提示增强 | 简单任务，1-2步 | 优化单agent提示词，不拆分不组队 |
| 2 | Skill直接复用 | 任务可由单个Skill完全解决 | 执行规划和Skill搜索后，直接调用匹配的Skill，无需组建Agent Teams |
| 3 | 计划+评审 | 中等/复杂任务（**默认**） | 出计划 → 用户确认 → 并行执行 → Review验收 |
| 4 | Lead-Member | 需要明确团队分工 | Leader协调分配，Member并行执行，通过TaskList协同 |
| 5 | 复合编排 | 复杂任务，无固定模式 | 动态组合上述场景，按阶段切换策略 |


**模型分工**（所有场景通用）：通过平台支持的模型选择能力按任务复杂度分配；Claude Code 可通过 Task 工具的 `model` 参数分配——`opus`处理复杂推理，`haiku`处理简单任务，`sonnet`处理常规任务。平台不支持模型选择时，不要写死模型承诺。

## 协作模式

| 模式 | 通信方式 | 适用场景 | Claude Code 启动方式 | Codex 启动方式 | OpenClaw / Cursor 启动方式 |
|------|---------|---------|---------|---------|---------|
| Subagent | 子agent → 主协调器单向汇报 | 并行独立任务 | `Task`工具 | 宿主提供的 subagent/custom-agent dispatch；不可用则主线程分阶段执行 | 平台 agent/background/workspace 能力；不可用则主线程分阶段执行 |
| Agent Team | 成员间可双向通信(SendMessage) | 需要协作的复杂任务 | `TeamCreate` + `Task(team_name)` | 多个独立 subagent + 主线程协调；仅在宿主暴露 agent I/O 时中途交互 | 平台 team/workspace/background-agent 能力；没有则降级 |

选择原则：任务间无依赖用Subagent（简单高效），任务间需要协调用Agent Team（功能更强但成本更高）。如果当前平台没有真正的 team bus，只能称为"多个独立 subagent + 主线程汇总"，不能伪装成成员间双向协作。

## 6阶段工作流（含强制规划和Skill搜索）

**重要说明**：阶段0和阶段1是**所有场景的强制前置步骤**，场景选择（1-5）只影响阶段2-5的执行方式。

### 阶段0：规划准备（Planning Setup）**【硬性标准 - 所有场景必经】**

**优先使用当前平台的 Skill 工具调用 planning-with-files**：
```
Skill(skill="planning-with-files")
```

跨平台适配：
- Claude Code：优先使用 `Skill(skill="planning-with-files")`
- Codex：若本地 skill/tool 可用则使用；否则使用平台计划工具或内联计划，并明确说明没有创建 planning files
- OpenClaw/Cursor：若本地 planning skill 可用则使用；否则维护平台可见计划记录

这将在项目目录创建三个核心文件（当 planning-with-files 可用时）：
- `task_plan.md` - 任务计划和阶段追踪
- `findings.md` - 研究发现和知识积累
- `progress.md` - 执行日志和进度记录

**关键规则**（规划文件创建后遵循）：
- 每个阶段开始前读取task_plan.md，完成后更新状态
- 每2次搜索/浏览操作后立即保存发现到findings.md
- 所有错误必须记录到task_plan.md的"Errors Encountered"表格
- 3次失败后升级给用户

> **铁律**：复杂任务不能没有计划就开始执行。若平台没有 planning-with-files，也必须使用等价计划记录或内联计划，不能跳过规划阶段。

### 阶段1：任务分析 + Skill发现（Discovery）**【硬性标准 - 所有场景必经】**

先质疑再执行：
- 需求不合理时主动挑战假设，建议更好的方案
- 区分"现在必须做"和"以后再说"，排除非核心范围
- 任务太大时建议更聪明的起点
- 先描述所需能力，再匹配 agent / skill / tool，避免 name-first 调度

输出任务总览：

| 字段 | 内容 |
|------|------|
| 任务目标 | [一句话描述] |
| 预期结果 | [具体交付物] |
| 验收标准 | [可量化的通过条件] |
| 范围界定 | [must-have vs add-later] |
| 当前平台 | [Claude Code / Codex / OpenClaw / Cursor / Unknown] |
| 预计Agent数 | [N个，建议≤5] |
| 选定场景 | [场景编号+名称] |
| 协作模式 | [Subagent/Agent Team/Degraded] |

**Skill完整回退链**（强制执行，不可跳过）：

对每个子任务执行以下3步fallback chain：

1. **本地Skill扫描**：
   - 读取当前运行时可见的 available skills / agents / tools / capability index
   - 提取每个skill的名称和触发词/描述
   - 将子任务关键词与skill触发词比对
   - 匹配成功 → 标注`[Skill: skill-name]`，进入阶段2直接调用

2. **外部Skill搜索**（本地无匹配时）：
   - 平台提供 Skill 工具时调用 find-skills：
   ```
   Skill(skill="find-skills", args="子任务关键词")
   ```
   - 平台没有 Skill 工具时，使用可用的外部能力搜索方式；没有搜索能力时向用户说明降级
   - 搜索到 → 向用户推荐：`npx skills add <owner/repo@skill-name> -g -y`
   - 用户确认安装 → 标注新skill，进入阶段2调用
   - 用户拒绝或平台无法安装 → 继续第3步


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KimYx0207/agent-teams-playbook](https://github.com/KimYx0207/agent-teams-playbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
