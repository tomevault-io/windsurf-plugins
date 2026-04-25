---
trigger: always_on
description: 本文档详细说明项目中所有 Agent 的配置、职责和使用方法。
---

# Agent 配置指南

本文档详细说明项目中所有 Agent 的配置、职责和使用方法。

## Agent 概述与自组织架构

本项目采用**六角色多 Agent 架构**，基于系统论、控制论和信息论设计，通过职责分离避免自评乐观偏差，同时维持高信噪比。

**架构演进**：当前的六个角色（Coordinator, Planner, Reviewer, Executor, Evaluator, Memory Curator）在系统设计中作为**角色分类（Role Categories/Lanes）**。针对不同场景的任务，每一角色下会不断沉淀出对应的**专家 Agent（Expert Agents）**，而基础的角色 Agent 则作为通用的 **Backup（兜底）** 存在。这体现了 Agent 系统的自组织和涌现能力。

**当前建模约定**：
- 六角色是**职责泳道**，不是六个固定实例。
- `configs/orchestration/agent-registry.yaml` 中的 `lanes` 只描述每条泳道的输入、输出、认领策略和选择规则。
- `members` 是 Team Roster，也是成员唯一真相源，描述 Backup、Expert、Candidate 三类成员。
- `lanes.member_source` 用于声明某条泳道应从哪一组成员中选择执行者。
- `agents` 保留为向后兼容模板视图，便于旧脚本或旧文档继续读取基础配置。

| 角色分类 (Role) | 职责 | 核心能力 | 工具权限 | Backup Agent | 专家 Agent 示例 |
|-------|------|----------|----------|--------------|----------------|
| **Coordinator** | 协调 | 意图识别、需求澄清、任务分发 | 只读 | `coordinator` | `api_coordinator` |
| **Planner** | 规划 | 需求分析、任务分解、方案设计 | 只读 | `planner` | `db_planner` |
| **Reviewer** | 计划审查 | 风险识别、方案评估、改进建议 | 只读 | `reviewer` | `sec_reviewer` |
| **Executor** | 执行 | 代码实现、测试编写、自测验证 | 读写+执行 | `executor` | `frontend_executor` |
| **Evaluator** | 评估 | 质量评估、需求验证、结论输出 | 只读+测试 | `evaluator` | `perf_evaluator` |
| **Memory Curator**| 记忆策展 | 上下文压缩、信息归档、信噪比控制 | 只读+执行 | `memory_curator` | `doc_curator` |

## 团队协作与自组织机制

基于系统进化，团队引入了以下三大核心机制，使得 Agent 团队能够真正地“自主运转”：

### 1. 团队名册与持久化队友 (Team Roster & Teammates)
- **持久化身份**：Agent 不再是调用完即销毁的临时对象（Subagent），而是长驻的、有明确身份和生命周期的队友（Teammate）。
- **名册与邮箱**：系统维护一份**团队名册（Team Roster）**。每个 Agent 都有自己独立的**邮箱（Inbox）**和**独立循环（Independent Loop）**，能够反复接手任务并保持上下文隔离。

### 2. 结构化团队协议 (Team Protocols)
- **协议消息**：团队协作不仅依靠自然语言，还引入了**结构化协议（ProtocolEnvelope）**。
- **请求与响应**：如“计划审批（plan_approval）”或“优雅关机（shutdown）”等关键协作，必须带有唯一的 `request_id`，并被记录在请求追踪表中（RequestRecord）。
- **状态机**：每个请求都具备明确的流转状态（如 pending / approved / rejected），确保多 Agent 协作时的过程可检查、状态可恢复。

### 3. 自主认领与执行 (Autonomous Claiming)
- **自主找活**：Agent 并非总是等待主控者点名分配任务。处于空闲（IDLE）状态的 Agent 会在每轮循环中先检查个人邮箱，随后**带着角色过滤条件**扫描公共任务板（Task Board）。
- **专家优先**：当任务板出现新任务时，带有对应领域标签的专家 Agent（如 `frontend_executor`）会优先触发认领（claim_task）。
- **原子化认领与兜底**：认领动作是原子的，且会被记录到事件日志（Claim Event Log）中。若任务长时间无专家认领，则由通用的 Backup Agent（如 `executor`）兜底认领。在认领后，Agent 会重新注入身份提示，带着明确的目标恢复工作（WORK）。

### 4. 候选专家沉淀与晋升 (Candidate Emergence)
- **模式提取**：Memory Curator 会从高质量成功任务中提取稳定的 prompt、toolchain、领域标签和交付模式。
- **候选生成**：当某类模式连续多次在同一泳道下成功复现时，系统会将其记录为 Candidate Expert，而不是立刻覆盖现有 Backup。
- **双重审查**：Candidate 需要经过 Reviewer 与 Evaluator 的结构化审批后，才能晋升为正式 Expert Agent。
- **渐进演化**：本阶段先在注册表和协议层沉淀这些元数据，后续再把自动晋升和真实 claim loop 接到运行时。

## 工作流程 (基于任务板与协议的异步流转)

```
┌────────────────────────────────────────────────────────────────────────┐
│                          用户提交意图                                  │
└────────────────────────────┬───────────────────────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────────────────────┐
│  Phase 0: 协调 (Coordinator Role)                                      │
│  ├─ 专家或 Backup Coordinator 自主扫描并认领意图处理任务                     │
│  ├─ 澄清模糊需求并输出结构化需求至 Task Board                               │
└────────────────────────────┬───────────────────────────────────────────┘
                             │ 发布 Requirement Task
                             ▼
┌────────────────────────────────────────────────────────────────────────┐
│  Phase 1: 规划 (Planner Role)                                          │
│  ├─ 专家 Planner (如 db_planner) 或 Backup Planner 自主认领需求任务         │
│  ├─ 设计方案并输出执行计划 (Execution Plan) 至 Task Board                   │
└────────────────────────────┬───────────────────────────────────────────┘
                             │ 发起 plan_approval 请求 (Protocol)
                             ▼
┌────────────────────────────────────────────────────────────────────────┐
│  Phase 2: 计划审查 (Reviewer Role)                                     │
│  ├─ 专家 Reviewer 或 Backup Reviewer 从邮箱收到 plan_approval 请求         │
│  ├─ 结构化响应: APPROVED / APPROVED_WITH_SUGGESTIONS / NEEDS_REVISION     │
└────────────────────────────┬───────────────────────────────────────────┘
                             │ 审查通过，发布 Execution Tasks
                             ▼
┌────────────────────────────────────────────────────────────────────────┐
│  Phase 3: 执行 (Executor Role)                                         │
│  ├─ IDLE 状态的专家 Executor (如 frontend_executor) 扫描并认领对应领域的 Task │
│  ├─ (若无专家认领) Backup Executor 兜底认领                                │
│  ├─ 执行代码实现与自测验证                                                  │
└────────────────────────────┬───────────────────────────────────────────┘
                             │ 提交实现结果，发布 Evaluation Task
                             ▼
┌────────────────────────────────────────────────────────────────────────┐
│  Phase 4: 评估 (Evaluator Role)                                        │
│  ├─ 专家/Backup Evaluator 认领评估任务                                     │
│  ├─ 输出评估结论: EXCELLENT / PASS / NEEDS_IMPROVEMENT                     │
└────────────────────────────┬───────────────────────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
┌─────────────────────────┐    ┌─────────────────────────┐

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cybernetix-lab/moss-harness](https://github.com/cybernetix-lab/moss-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
