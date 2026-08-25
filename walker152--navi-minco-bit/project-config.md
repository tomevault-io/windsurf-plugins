---
trigger: always_on
description: 本文件是本仓库中 AI Agent、Codex、Claude、superpower skill、多 agent 工作流的统一行为规范。
---

# AGENTS.md

# 项目 Agent 行为准则

本文件是本仓库中 AI Agent、Codex、Claude、superpower skill、多 agent 工作流的统一行为规范。  
所有代码修改、审计、重构、调参、文档化任务都必须优先阅读并遵守本文件。

本仓库是 RoboMaster 哨兵机器人比赛代码仓库，包含行为树、自主决策、planner、controller、地图、里程计、点云、裁判系统通信、launch、参数文件等模块。仓库中存在已经经过比赛验证的逻辑，任何修改都必须以“保持可复现、最小改动、可审计”为第一原则。

---

## 0. 最高优先级限制：编译构建限制

**禁止在未经用户明确允许的情况下执行任何编译或构建操作**，包括但不限于：

- `colcon build`
- `catkin build` / `catkin_make`
- `cmake` 构建
- `make` / `ninja`
- `cargo build` / `cargo check`
- `npm run build` / `yarn build`
- 任何其他会触发代码编译、链接的命令

在执行任何编译构建命令之前，必须先向用户确认并获得明确许可。

允许在未确认前执行的低风险检查包括：

- `grep` / `rg`
- `find`
- `ls`
- `sed` / `cat`
- `python3` 静态解析脚本
- XML 语法解析
- YAML 语法解析
- git diff / git status
- 不触发编译、链接、生成代码的静态检查

如果不确定某条命令是否会触发构建，必须先询问用户。

## 0.1 最高优先级限制：禁止版本控制提交操作

**禁止执行任何会改变 Git 历史、Git 索引、远端仓库、分支或标签状态的提交类操作**，无论用户是否要求修改代码，均不得由 Agent 主动提交。

禁止命令包括但不限于：

- `git add`
- `git commit`
- `git commit --amend`
- `git push`
- `git tag`
- `git merge`
- `git rebase`
- `git cherry-pick`
- `git revert`
- `git reset --hard`
- `git checkout -- <file>` / `git restore <file>` 等会覆盖工作区修改的命令
- `gh pr merge`、`gh release create` 等任何会产生提交、合并、发布或远端变更的命令

允许的只读 Git 检查包括：

- `git status`
- `git diff`
- `git log`
- `git show`

Agent 只负责修改工作区文件和说明变更，不负责暂存、提交、推送、打标签或合并。  
即使任务已经完成，也只能提醒用户自行检查和提交，不能代替用户执行任何提交操作。

## 0.2 最高优先级限制：禁止 writingplans / 计划说明工具

**禁止使用 `writingplans`、Superpower writing plan、计划说明 skill 或类似工具来生成计划、改造计划、测试计划或阶段说明。**

执行方式要求：

1. 用户与 Agent 已经商讨清楚需求和边界后，直接修改代码或文件。
2. 不要在正式修改前额外输出长篇“计划说明”“改造计划说明”“测试计划说明”。
3. 不要新建 `writingplans` 相关文件。
4. 不要把计划说明作为修改前的阻塞步骤。
5. 必要的任务边界、风险、检查项可以写入最终改造记录或最终回复，但不能调用 writingplans 工作流。
6. 多 Agent 流程只有在用户明确要求时使用，且不得用 writingplans 替代 Explorer / Modifier / Auditor 的实际检查。

允许保留简短的执行摘要和最终检查结果，但不得以 writingplans 形式展开。  
核心原则是：**商讨完毕后直接按约束修改代码，修改完成后再给出摘要、检查结果和剩余风险。**

---

## 1. 项目背景

本仓库包含但不限于以下模块：

- 行为树与自主决策：`bt_manager`、行为树 XML、blackboard、裁判系统通信。
- 规划器：MINCO planner、路径搜索、轨迹优化、重规划逻辑。
- 控制器：MPC controller、controller_server、速度输出、参数切换。
- 地图与感知：ROGMap、ProjectionLayer、ESDF、地形投影、动态障碍处理。
- 里程计与点云：Point-LIO / Batch-LIWO、Livox 驱动、完整点云输出、去畸变、odom。
- ROS 2 通信：component container、intra-process、QoS、topic、launch、参数文件。
- 比赛策略：前哨站、冲家、补给、复活、姿态、小陀螺、隧道、台阶、资源兑换。
- 文档与实验材料：论文、申请材料、性能记录、CSV、图片、演讲稿等。

这是比赛代码，不是单纯实验仓库。  
不要为了“理论更优”而随意改变比赛验证过的逻辑。

---

## 2. 总原则

所有 agent 必须遵守：

1. 不破坏比赛验证过的主逻辑。
2. 不进行无关重构。
3. 不顺手删除历史逻辑、注释分支或旧方案，除非用户明确要求。
4. 不引入大量新变量、新函数、新模块。
5. 不改变已有 topic、frame、参数名、blackboard key，除非用户明确要求。
6. 不改变模块边界，除非这是本次任务目标。
7. 不添加高频 debug 日志。
8. 不添加无用统计。
9. 不擅自修改 launch 组合方式、QoS、timer、callback group。
10. 不擅自修改比赛参数默认值。
11. 不把未验证推断写成确定事实。
12. 每次非平凡修改必须留下改造记录。

如果用户没有明确要求重构，应优先选择：

```text
最小修改 > 局部修补 > 结构整理 > 大规模重构
```

---

## 3. 用户确认时使用多 Agent 工作流

每次非平凡代码修改，如果用户明确说明使用三阶段流程：

```text
Explorer Agent → Modifier Agent → Auditor Agent
```

多 agent 的目标不是让多个 agent 同时乱改，而是通过角色隔离降低误判风险。  
多 agent 记录不得使用 writingplans 生成，也不得以计划说明替代真实仓库检查。

### 3.1 Explorer Agent：探索与事实确认

Explorer Agent 只读仓库，不修改代码。

职责：

1. 找到与用户需求相关的源码、头文件、XML、launch、yaml、参数文件。
2. 梳理调用链、数据流、topic、blackboard key、参数来源。
3. 区分“当前生效逻辑”和“注释/废弃/历史逻辑”。
4. 标记比赛验证逻辑、隐式依赖、潜在冲突。
5. 给 Modifier Agent 明确修改边界。
6. 把探索结果写入改造记录。

必须检查：

- XML 或 launch 中实际启用的路径。
- cpp/hpp 中对应节点是否注册、是否被调用。
- blackboard key 的读写者。
- ROS topic 的发布者、订阅者、QoS、频率限制。
- 参数是否来自 yaml、declare_parameter、硬编码或 launch override。
- 是否有 static、全局变量、类成员状态影响多实例行为。
- 是否存在同名节点、重复分支、历史注释误导。
- 是否有用户明确要求不能动的模块。

Explorer Agent 禁止修改文件。

### 3.2 Modifier Agent：最小修改

Modifier Agent 只能根据 Explorer Agent 的记录和用户目标修改代码。

职责：

1. 做最小范围修改。
2. 保留原有行为优先级。
3. 不做用户未要求的清理、重命名、重构。
4. 修改后更新必要注释、参数说明、XML name 或记录文件。
5. 如果发现 Explorer 结论不完整，先补充记录，不要直接扩大修改范围。

禁止：

- 顺手改其他模块。
- 顺手加调试统计。
- 顺手改参数默认值。
- 顺手删注释分支。
- 顺手重构类结构。
- 顺手改 topic 名、frame_id、QoS、blackboard key。
- 顺手把逻辑改成“理论上更优”的版本。

### 3.3 Auditor Agent：审计与验收

Auditor Agent 不相信 Modifier 的自述，必须重新检查。

职责：

1. 查看 diff。
2. 重新 grep 关键变量、节点、topic、参数。
3. 检查是否越界修改。
4. 检查是否破坏原有优先级和数据流。
5. 运行用户允许范围内的静态检查。
6. 若需要编译构建，必须先向用户确认。
7. 给出 `PASS` 或 `NEEDS_FIX`。

Auditor 发现问题后，不要大范围自行改造。  
应记录问题，并让 Modifier 进行针对性修复。

---

## 4. 改造记录要求

每次非平凡任务默认需要在修改完成后新建或更新记录文件：

```text
docs/ai_refactor_records/<YYYYMMDD>_<task_name>.md
```

如果用户指定记录文件，则使用用户指定路径。  
改造记录是**事后审计材料**，不是 writingplans，也不是修改前计划说明。  
不得为了撰写计划、改造计划或测试计划而中断已经确认的代码修改流程；用户与 Agent 商讨完毕后，应直接进行修改，再在完成后补充必要记录。  
如果用户明确要求本次不写改造记录，则以用户要求为准。

记录文件必须包含：

```md
# <任务名称> 改造记录

## User Intent

用户原始目标和关键约束。

## Scope

本次允许修改的范围。

## Out of Scope

本次明确不处理的内容。

## Explorer Findings

### Files inspected

### Active logic path

### Data flow

### Risk notes

### Recommended modification boundary

## Modifier Changes

### Files changed

### Key changes

### Behavior preserved

### Behavior intentionally adjusted

### Notes

## Auditor Review

### Checks performed

- [ ] 关键路径检查
- [ ] diff 检查
- [ ] grep 检查
- [ ] XML / launch / yaml 检查
- [ ] 用户允许范围内的测试或静态检查
- [ ] 如需构建，已取得用户明确许可

### Issues found

### Final result

PASS / NEEDS_FIX
```

最终回答用户时，必须提供：

1. 修改摘要。
2. 修改文件列表。
3. 检查结果。
4. 改造记录路径。
5. `PASS` 或 `NEEDS_FIX`。
6. 如果未执行构建，必须说明“未执行构建，因为 AGENTS.md 禁止未授权构建”。

---

## 5. 修改前必须建立任务边界

修改前必须在内部明确任务边界；记录文件可在修改完成后补充。不得使用 writingplans 输出独立计划说明：

### 5.1 本次修改属于哪个模块

可选：

- 行为树 / 自主决策
- planner
- controller
- map / ROGMap
- odom / pointcloud
- launch / communication
- referee / blackboard
- parameter / yaml
- 文档 / 实验记录

### 5.2 本次修改目标是什么

可选：

- bug 修复
- 策略调整
- 性能优化
- 通信链路改造
- 参数暴露

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Walker152/navi_minco_bit](https://github.com/Walker152/navi_minco_bit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
