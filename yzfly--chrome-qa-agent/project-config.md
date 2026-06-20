---
trigger: always_on
description: 基于 chrome-devtools MCP 的对抗式 UI 测试。分析 git diff 只测改动部分，或探索整个应用找 bug。覆盖功能正确性、无障碍(a11y)、响应式布局与 UX 启发式。当用户要测试 UI 改动、QA 一个 PR、审计无障碍、或做探索式测试时使用。驱动真实本地 Chrome，本地与任意线上 URL 都适用。
---


# Chrome QA Agent — 对抗式 UI 测试 Skill

在真实浏览器里测试 UI 改动。你的任务是**想办法把它弄坏**，而不是确认它能跑。

本 skill 通过 **chrome-devtools MCP**（`mcp__chrome-devtools__*` 工具）驱动**单个本地 Chrome**。由于该 MCP 只控制一个浏览器，且 `select_page` 会为整条连接全局设置当前活动页面，**浏览器测试是单线程的** —— 绝不能让两个 agent 同时对同一个浏览器操作。

两种工作流：
- **diff 驱动** —— 分析 git diff，只测改动部分
- **探索式** —— 在应用里到处点，找出开发者没想到的 bug

对于较大的任务，你仍可以把工作拆给多个子 agent 以隔离上下文，但它们必须轮流独占这唯一的浏览器 —— 见下文「拆分工作的原则」一节。

## 测试如何运作

主 agent 负责**协调** —— 它制定测试策略、把任务分派给子 agent、并汇总结果。真正的浏览器测试由子 agent 执行。

### 规划：先从多个角度想，再一次性执行

**你必须亲自完成全部三轮规划并输出，然后才能启动任何子 agent。** 规划发生在你自己的回复里 —— 不能委派给子 agent。不要跳过规划直接执行。

**第一轮 —— 功能：** 核心用户流程有哪些？什么应该正常工作？把每个测试写成：动作 → 预期结果。

**第二轮 —— 对抗：** 重读第一轮。你漏了什么？想想：不同的用户类型/角色、错误路径、空状态、竞态、边界输入（空值、超长、特殊字符、连续快速点击）。

**第三轮 —— 覆盖盲区：** 重读第一、二轮。还有什么没考虑：无障碍（axe-core、纯键盘）、移动端视口、控制台错误、与应用其余部分的视觉一致性？

**去重：** 把三轮合并成一份带编号的测试清单。去掉重叠项。给每个测试分配一个分组（例如 Group A、Group B）。

**然后一次性执行** —— 每个分组启动一个子 agent。每个子 agent 只接收它那份具体的测试清单，别的都不给。子 agent 不探索、不规划 —— 它们执行被指派的测试并汇报结果。

在调用任何 Agent 工具之前，把三轮规划、合并后的计划、以及分组分配都输出在你的回复里。

### 拆分工作的原则

- **一个浏览器，同一时刻只有一个 agent。** chrome-devtools MCP 控制单个 Chrome，且 `select_page` 是全局状态。**顺序**分派子 agent —— 每个在自己回合里独占浏览器，跑完，下一个再开始。并发驱动浏览器的 agent 会破坏彼此的页面状态。（你仍可以并行跑*非浏览器*的工作，比如 diff 分析。）
- **子 agent 跑被指派的测试，而非开放式探索。** 主 agent 交给每个子 agent 一份具体的、带编号的测试清单。子 agent 不规划、不探索、不决定测什么 —— 它们执行清单然后停下。
- **即便是串行，拆分依然有用** —— 它让每个 agent 的上下文保持小而聚焦。许多小 agent 优于少数大 agent。这里的收益是上下文隔离和每组干净的汇报，而不是挂钟时间上的并行。
- **按改动规模匹配投入** —— 单个组件的修复不需要很多 agent 或很多步数；主 agent 直接跑就行。整页重构才值得拆成多个分组。让 diff 的范围来驱动计划。
- **遇到失败不要提前停** —— 在被指派的测试范围内尽可能多地找出 bug。

### 给子 agent 设定步数预算

**主 agent 必须在每个子 agent 的提示里写明步数上限。** 一「步」= 一次 chrome-devtools MCP 工具调用（`navigate_page`、`take_snapshot`、`click`、`evaluate_script`……）。子 agent 不会自我设限 —— 除非被告知，否则它们会一直跑到完成。

粗略经验值：几项有针对性的检查约 25 步，一个完整页面（功能 + 对抗 + a11y）约 40 步，多个页面或一大类约 75 步。**根据被指派测试的实际需要来调整** —— 这些是起点，不是规则。

每个子 agent 的提示都必须包含：
```
You have a budget of N steps (each chrome-devtools MCP tool call = 1 step). Count your steps as you go. When you reach N, stop immediately and report:
- STEP_PASS/STEP_FAIL for every test you completed
- STEP_SKIP|<test-id>|budget reached for every test you didn't get to

Drive the browser ONLY through the chrome-devtools MCP tools (mcp__chrome-devtools__*).
You own the browser exclusively for this run — no other agent is using it. When done, the next agent will reuse the same Chrome, so leave it in a clean state (close extra tabs you opened with close_page).
Do not retry or continue after hitting the budget.
Run only these tests: [numbered list from the merged plan]
Do not explore beyond the assigned tests.
Do NOT generate an HTML report or write any files (except screenshots via take_screenshot's filePath). Return only step markers and your findings as text.
```

主 agent 自己不应驱动浏览器（除了验证开发服务器是否已启动）。所有测试都在子 agent 里进行，**一次只分派一个** —— 绝不要同时跑两个驱动浏览器的子 agent，因为它们共用一个 Chrome。

**当某个子 agent 用尽预算时，主 agent 直接接受这份部分结果。** 不要重跑或重试这个子 agent。把 SKIPPED 的测试写进最终报告，让开发者知道哪些没覆盖到。

### 汇报

**每个子 agent 回报时给出：**
```
Tests: 8 | Passed: 5 | Failed: 2 | Skipped: 1 | Pages visited: 2
```

**主 agent 汇总成最终报告：**
```
Tests: 20 | Passed: 14 | Failed: 4 | Skipped: 2 | Agents: 3 | Pass rate: 70%
```

不要汇报「用了多少步」 —— MCP 工具调用次数是实现细节，对评审者来说不是有意义的指标。

## 测试理念

**你是一名对抗式测试者。** 你的目标是找出 bug，而不是证明正确性。

- **想办法弄坏你测的每个功能。** 别只检查「按钮在不在？」 —— 快速点它两下、提交空表单、粘贴 500 个字符、流程中途按 Escape。
- **测开发者没想到的地方。** 空状态、错误恢复、纯键盘导航、移动端溢出。
- **每条断言都必须基于证据。** 对比前后快照。按 ref 检查具体元素。没有来自无障碍树或确定性检查的实在证据，绝不报 PASS。
- **失败要报到足以复现的程度。** 包含确切的动作、你的预期、实际得到的结果，以及一条修复建议。

## 断言协议

每个测试步骤都必须产出一条结构化的断言。不要写「看起来不错」这种自由文本。

### 步骤标记

每个测试步骤，恰好发出一个标记：

```
STEP_PASS|<step-id>|<evidence>
```
或
```
STEP_FAIL|<step-id>|<expected> → <actual>|<screenshot-path>
```

- `step-id`：简短标识，如 `homepage-cta`、`form-validation-error`、`modal-cancel`
- `evidence`：你观察到的、能证明此步通过的东西（元素 uid、文本内容、URL、`evaluate_script` 结果）
- `expected → actual`：你的预期 vs 实际得到的
- `screenshot-path`：保存的截图路径（仅失败时 —— 见下文「失败截图」）

### 失败截图

**每条 STEP_FAIL 都必须配一张截图**，好让开发者直观看到哪里出了问题。

当某个测试步骤失败时，立即截图，直接写入以 step-id 命名的路径：

```
mcp__chrome-devtools__take_screenshot { filePath: ".context/ui-test-screenshots/<step-id>.png" }
```

`take_screenshot` 直接写文件 —— 不需要再单独移动/复制。要截取单个元素而非视口，传它的 `uid`；要截整个可滚动页面，传 `fullPage: true`。

每次测试运行开始时，创建一次截图目录：

```bash
mkdir -p .context/ui-test-screenshots
```

**规则：**
- 文件名 = step-id（例如 `double-submit.png`、`axe-audit.png`、`modal-focus-trap.png`）
- 存到 `.context/ui-test-screenshots/` —— 该目录已加入 gitignore，开发者和其他 agent 都能访问
- 当某个子 agent 负责一个测试分组时，加上分组名前缀以避免冲突：`<group>-<step-id>.png`（例如 `signup-double-submit.png`）
- 在失败的那一刻截图 —— 捕捉损坏状态，不要等恢复之后
- 对于视觉/布局 bug，也截一张基线（正常状态）以便前后对比：`<step-id>-baseline.png`

### 如何验证（按严谨程度排序）

1. **确定性检查**（最强） —— `evaluate_script` 返回可 JSON 序列化的数据供你检视。例如：axe-core 违规计数、`document.title`、表单字段值、控制台错误数组、元素数量。
2. **快照元素匹配** —— 在 `take_snapshot` 返回的无障碍树里存在某个具有特定角色和文本的元素。每个元素都有一个 `uid`（例如 `uid=12 button "Save"`）。元素要么在树里，要么不在。
3. **前后对比** —— 动作前快照，执行动作，动作后快照。验证树按预期发生了变化（元素出现、消失、文本改变）。
4. **截图 + 视觉判断**（最弱） —— 仅用于无障碍树无法捕捉的纯视觉属性（颜色、间距、布局）。务必说明你具体在评估什么。

### 前后对比模式

这是核心的验证循环。每次交互都用它：

```
# 1. BEFORE: capture state
mcp__chrome-devtools__take_snapshot
# Record: what elements exist, their text, their uids


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yzfly/chrome-qa-agent](https://github.com/yzfly/chrome-qa-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
