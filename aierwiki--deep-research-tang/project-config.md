---
trigger: always_on
description: 唐氏深度研究法（Tang Deep Research）。迭代式深度研究工作流，通过多轮「发散→收敛」循环对复杂问题进行系统化深入分析。适用于投资机会挖掘、问题本质探究、技术调研、方案选型等需要深度思考的任务。触发词：唐氏深度研究、深度研究、深度调研、迭代研究、深入分析、投资研究、deep research、iterative research。
---


# 唐氏深度研究法（Tang Deep Research）

由算法工程师唐玉宾提出的一种迭代式深度研究工作方法。核心思路：**发散 → 收敛 → 发散 → 收敛 → … → 最终结论**。

每一轮探索都会带来新的信息、新的线索、新的视角，这些新发现又会驱动下一轮更有针对性的发散探索。通过反复的发散-收敛循环，不断拓展认知边界，最终形成深入且全面的结论。

这套方法的精髓不是一开始就把整条研究路线规划得非常死板，而是在大方向正确的前提下，根据研究过程中不断累积的新信息随机应变，动态调整后续的研究方向、任务设计和推进方式。

---

## 何时使用

- 挖掘投资机会——发现被忽视的标的、分析行业趋势、评估风险与收益
- 探究问题本质——透过表象找到根本原因，避免停留在浅层解释
- 技术调研、方案选型——对比多个方案，权衡利弊
- 任何需要多维度、多层次深入分析的复杂问题

**不适合**：能直接回答的简单问题。

---

## 研究深度

深度研究支持 1-100 级的研究深度，深度级别决定了发散-收敛循环的迭代轮数。

- **用户指定深度**：用户可以明确要求研究深度，例如"深度 7 级研究"意味着要执行 7 轮发散-收敛迭代。深度越高，研究越充分，但耗时也越长。**这是硬约束**：只要用户明确指定了研究深度、研究轮次或等价的轮数要求，Agent 就必须研究够对应轮数才能停止，不能因为主观觉得“已经差不多了”“信息已经够了”就提前结束。
- 这里的“研究够对应轮数”指的是：最后一轮也仍然要完整经历“任务拆解 → 执行探索 → 阶段总结 → delta 产出”的闭环。**最终回答用户的问题不是最后一轮本身，而是所有轮次完成之后新增的一个独立综合阶段。**
- **自动判定深度**：如果用户没有指定深度，根据问题的复杂程度和模糊程度动态决定。边界清晰的具体问题 2-3 轮足够，开放性的复杂问题可能需要 7-100 轮。

深度研究的价值恰恰在于足够多轮次的深入挖掘。不要害怕迭代轮数多，重要的是每一轮都能有新发现、新线索，推动研究不断深入。
---

## 研究流程

### 1. 初始化

在当前工作目录下创建一个归档文件夹，用于存放整个研究过程的所有文件。

如果当前环境是 OpenClaw，则**必须先启动 deep-research 会话**，而不是直接手工创建目录。原因是：它不只会初始化归档，还会把“当前这一次研究”绑定成 OpenClaw 的活跃 deep-research 会话，让 runtime guard 只对这一次研究生效。

OpenClaw 中如果存在 `deep_research_session` 工具，**必须优先调用它**：

```json
{
  "action": "start",
  "topic": "<topic-slug>",
  "question": "<用户问题>",
  "target_depth": <N>,
  "depth_mode": "auto"
}
```

轮次推进和最终收口也优先继续走同一个工具，而不是手工改 `00_meta.json`：

```json
{
  "action": "advance-round"
}
```

适用时机：当前轮的 `03_round_summary.md` 和 `04_delta_report.json` 已经写完，准备进入下一轮或确认已完成全部轮次。这个动作会原子完成：

- 校验当前轮
- 更新本轮通过后的 meta 状态
- 若还没到目标轮次，则自动起好下一轮骨架

全部轮次完成且 `final_report.md` 已写成真实内容后，再调用：

```json
{
  "action": "finalize"
}
```

这个动作会先校验全归档，再把研究状态原子收口到 `completed`。

如果研究中途停止、换了聊天会话，或不确定下一步该做什么，优先调用：

```json
{
  "action": "recover"
}
```

它会读取当前活跃研究归档、运行检查器，并返回下一步最小修复或继续建议。OpenClaw runtime guard 默认是 lite 模式，主要依赖 `advance-round` / `finalize` / `recover` 这些 checkpoint，而不是在每个工具调用上做强拦截。

只有在该工具不存在或不可用时，才允许退回到脚本方式：

```bash
python scripts/openclaw_deep_research_session.py start \
  --topic <topic-slug> \
  --question "<用户问题>" \
  --target-depth <N> \
  --depth-mode <auto|user-specified>
```

如果用户明确要求继续某个已有研究，而不是新开一轮，则用：

```bash
python scripts/openclaw_deep_research_session.py activate \
  --research-dir /absolute/path/to/research_xxx
```

强约束：

- 如果 `deep_research_session` 工具或会话脚本调用失败，**禁止**退化成手工 `mkdir research_xxx` 再继续研究
- 正确做法是：读取错误，修复原因，重试会话启动
- 只有会话真正启动/激活成功后，才允许写 `00_meta.json`、`round_N/*`、`final_report.md`
- OpenClaw 中优先使用 `deep_research_session` 的 `advance-round` / `finalize` 完成生命周期跳转，不要手工编辑 `00_meta.json` 推进轮次或收尾

在 OpenClaw 中，启动或激活研究会话后，**进入任何真实研究动作前必须先读取以下文件**，并以它们作为唯一权威协议：

- `templates/01_seed_clues.json`
- `templates/02_task_registry.json`
- `templates/04_delta_report.json`
- `scripts/check_deep_research_archive.py`
- `scripts/deep_research_state_machine.py`

约束如下：

- `SKILL.md` 只负责解释方法论，不是 schema 权威来源
- 真实字段名、合法状态值、轮次流转规则，必须以模板与脚本中的实现为准
- 不允许把 memory、旧归档、历史修复记录、旧实验 run 当作 schema authority
- 如果 memory 或旧案例与当前模板/脚本冲突，必须忽略 memory，服从当前仓库实现

**文件夹命名**：`research_<日期>_<主题关键词>/`，例如 `research_20260411_api-gateway/`

初始化阶段不是可选建议，而是**继续研究前的前置条件**。Agent 必须先完成以下归档骨架，才能进入任务拆解：

- `00_research_brief.md`：记录用户原始问题、研究目标、约束条件、预期交付物
- `00_meta.json`：记录机器可读元数据，至少包含 `topic`、`target_depth`、`depth_mode`、`current_round`、`status`
- 若仓库中存在 `scripts/check_deep_research_archive.py`，初始化完成后应立即运行一次检查，确认归档骨架有效

如果仓库中存在 `scripts/init_deep_research_archive.py`，应优先调用该脚本创建研究目录和模板文件，而不是手工拼装目录结构。只有在脚本不存在或确实无法执行时，才允许手工初始化。

`00_meta.json` 推荐结构如下：

```json
{
	"topic": "api-gateway",
	"original_question": "用户的原始问题",
	"target_depth": 5,
	"depth_mode": "user-specified",
	"current_round": 0,
	"status": "initialized"
}
```

### 2. 问题拆解（发散阶段）

根据用户的需求（以及上一轮总结中发现的新线索），把问题拆解成若干个可独立执行的探索任务。

这里要避免在研究一开始就把后面所有轮次的任务一次性规划死。深度研究只需要先把当前这一轮拆好，后续轮次应该由当前轮的发现来决定。

**拆解时思维要足够发散。** 不是只有"确定需要"的任务才值得做。任何有一点可能性的方向、甚至看起来关联不大的探索角度，都可以作为任务加进来。这一步的目标是尽可能拓宽视野，防止研究过早收敛到浅显或片面的状态。

可以刻意加入一些"野生"任务——看似边缘甚至随机的探索方向。很多时候，意外的发现恰恰来自这些非常规的探索路径。主线任务保证研究的基本覆盖，野生任务负责制造惊喜和突破。

从这一阶段开始，Agent 必须先写归档文件，再开始任何搜索、抓取、读文件、运行程序等探索动作。每一轮至少产出以下文件：

- `round_N/01_seed_clues.json`：本轮切入线索。第 1 轮来自原始问题，第 N 轮（N > 1）必须显式引用上一轮 `04_delta_report.json` 中的线索 ID
- `round_N/02_task_registry.json`：任务登记表，是本轮唯一合法的任务清单

拆解要点：
- 每个任务应该是独立的，可以并行执行
- 每个任务都是一个完整的探索子任务，不等于只搜一个 query；它可以包含搜索、读文件、写程序、运行程序、抓取网页、调用工具、交叉验证等完整执行动作
- 任务数量根据问题复杂度决定，一般 8-20 个
- 每个任务要有清晰的目标——要找出什么、搞清楚什么
- 不要自我审查过度——"这个方向可能没用"不是拒绝探索的理由
- 把任务计划写入归档文件夹

`02_task_registry.json` 中的每个任务至少要包含以下字段：

- `task_id`：本轮唯一 ID，例如 `R01-T03`
- `title`：任务标题
- `task_type`：`exploratory`、`verification`、`counterevidence`、`wildcard` 之一
- `research_dimension`：研究维度，必须尽量避免重复
- `key_question`：本任务唯一要回答的问题
- `planned_actions`：至少 3 个动作，禁止只写单个搜索动作
- `expected_evidence`：预期获得的证据类型
- `depends_on`：默认必须为空；若非空，说明拆解失败，应重构任务
- `report_path`：本任务结果文件路径

任务独立性的最低要求：

- 同一轮任务的 `key_question` 不能重复
- 同一轮任务的 `report_path` 必须一一对应
- `depends_on` 必须为空数组；不允许把前一个任务的输出作为后一个任务的前置条件
- 若大部分任务落在同一 `research_dimension`，说明拆解过窄，必须重写登记表

如果检查器对 `02_task_registry.json` 给出失败结果，Agent 必须先重写任务登记表并重新检查，禁止带着失败结果进入执行探索。

### 3. 执行探索

逐个执行拆解出来的探索任务。如果 Agent 框架支持 subagent，优先并行执行以提高效率。

这里的“执行探索”不是机械地对每个任务发一个搜索请求就算完成。一个探索任务本身可能就是复杂执行单元：需要多次搜索、读取资料、编写并运行代码、抓取网页、调用外部工具、做交叉验证，甚至中途根据发现调整该子任务内部的执行路径。评价标准是这个任务有没有把该方向真正探明，而不是有没有发出过一个 query。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aierwiki/deep-research-tang](https://github.com/aierwiki/deep-research-tang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
