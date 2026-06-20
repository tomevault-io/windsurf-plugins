---
trigger: always_on
description: Name: ai-office-landing
---

Name: ai-office-landing
Description: AI Office — Living Brief 协作循环(v2 对话式交互)
Entry Point: true

---

Skill Contents
- [Overview](#overview)
- [Usage](#usage)
- [Workflow](#workflow)
- [State Management](#state-management)
- [Phase 0: Design Reference Collection + Web Search](#phase-0-design-reference-collection--web-search)
- [Phase 1: Initial Interview + Iterative Refinement](#phase-1-initial-interview--iterative-refinement)
- [Phase 2: Style Tokens & Task Generation](#phase-2-style-tokens--task-generation)
- [Phase 3: Sequential Execution with Q&A](#phase-3-sequential-execution-with-qa)
- [Phase 4: Critic Review + Clarification Loop](#phase-4-critic-review--clarification-loop)
- [Phase 5: Final Delivery](#phase-5-final-delivery)
- [Resume & Recovery](#resume--recovery)
- [Cost Expectations](#cost-expectations)

---

## Overview

AI Office v2 是一个**对话驱动**的 Living Brief 协作循环：

**核心升级：**
- **对话式交互**: Executor 可在执行中提问，不必一次性想清楚
- **增量交付**: 每个阶段完成后展示预览，用户确认后再继续
- **问题缓冲**: Agent 的问题统一收集，批量问用户
- **双向通信**: 从"批处理"升级为"请求-响应"模式

**保留下来的优点:**
- Living Brief 作为单一事实来源
- Critic 独立审查，不继承主控上下文
- 风格令牌自动转译(hex/px)
- 断点续跑

New Constraints:
- `[QUESTION: ...]` 标记 Agent 需要用户澄清的地方
- `[USER_INPUT: ...]` 标记用户追加的信息
- `state.json` 持久化当前进度，跨 session 可恢复

---

## Usage

在任意项目目录下触发：

```bash
ai-office-landing [options]
```

**Options:**
- `--resume`: 从 state.json 恢复上次的对话式工作流
- `--auto-continue`: 自动继续，不暂停等待用户确认（适合批处理场景）
- `--human-critic`: 启用人工 Critic 模式
- `--cost-saving`: 启用成本节省模式，自动将 MEDIUM/LOW 角色路由到 Kimi/DeepSeek
- `--adapter <name>`: 强制所有 Executor 使用指定 adapter (claude-agent|kimi-api|deepseek-api)

**Adapter 路由 (v2.4 新增):**

Executor 任务通过 `adapters/route.sh` 自动路由到最优 adapter：

| 角色 | 质量等级 | 默认 Adapter | 成本节省模式 |
|------|---------|-------------|------------|
| Interviewer, Critic, Integrator | HIGH | claude-agent | claude-agent (不可降级) |
| Frontend | MEDIUM-HIGH | claude-agent | deepseek-coder |
| Copywriter, Designer | MEDIUM | claude-agent | kimi-api |
| SEO | LOW | claude-agent | kimi-api / deepseek-api |

环境变量配置：
```bash
export KIMI_API_KEY="your-key"        # 启用 Kimi adapter
export DEEPSEEK_API_KEY="your-key"    # 启用 DeepSeek adapter
export COST_SAVING_MODE=true          # 自动路由到便宜的 adapter
export AI_OFFICE_ADAPTER=kimi-api     # 强制指定 adapter (HIGH 角色除外)
```

**对话式示例：**
```bash
# 启动对话式工作流
ai-office-landing

# 过程中的交互
[Phase 1 完成] 
✓ 保存了 5 个维度的信息
✓ 但 Designer 想问：是否有品牌 logo 图片？

[Phase 3 - Copywriter 完成]
✓ 生成了 Hero/Features/FAQ/CTA 文案
✓ 但 Frontend 想问：是否接受 AI 生成的主视觉图？

[Phase 4 - Critic 审查]
⚠️ HIGH: Copy.md 使用了 "爆款"（在禁用词列表）
→ 需要你的修正: 删除或替换
```

---

## Workflow

```
Phase 0: Initialize
  ↓ 创建 ai-office/ 目录, state.json

Phase 1: Initial Interview
  ↓ AskUserQuestion 批量问 5 维度
  ↓ 生成 brief.md v1
  ↓ *可选* 收集阶段: 其他 Agent 可追加问题
      ↓ 如果有 [QUESTION], 问你 → 更新 brief.md → 回到 Checkpoint
      ↓ 如果没有 → 进入 Phase 2

Phase 2: Style Tokens & Tasks
  ↓ 生成 style-tokens.md, tasks.md
  ↓ *可选* 收集阶段
      ↓ 如果有 [QUESTION] → 问你 → 更新 → 回到 Checkpoint
      ↓ 如果没有 → 进入 Phase 3

Phase 3: Sequential Execution (逐个执行)
  ↓ Task 1: Copywriter → copy.md
      ↓ 检查是否有 [QUESTION]
      ↓ 有 → 问你 → 修正 → 回到 Task 1
      ↓ 无 → 继续 Task 2
  ↓ Task 2: Designer → design-spec.md
      ↓ 检查是否有 [QUESTION]
      ↓ 有 → 问你 → 修正 → 回到 Task 2
      ↓ 无 → 继续 Task 3
  ↓ Task 3: Frontend → index.html
      ↓ 检查是否有 [QUESTION]
      ↓ 有 → 问你 → 修正 → 回到 Task 3
      ↓ 无 → 继续 Task 4
  ↓ Task 4: SEO → meta.md
      ↓ 检查是否有 [QUESTION]
      ↓ 有 → 问你 → 修正 → 回到 Task 4
      ↓ 无 → 进入 Phase 4

Phase 4: Critic Review
  ↓ 独立 Critic 审查所有 outputs
  ↓ 生成 critique.md
  ↓ *可选* 如果有 HIGH/CRITICAL
      ↓ 收集所有问题 → 统一问你 → 修正 → 回到 Phase 3
      ↓ 如果都通过 → 进入 Phase 5

Phase 5: Final Delivery
  ↓ 整合所有交付物
  ↓ 清除 state.json
```

---

## State Management

### State File Location

`ai-office/state.json` - 存储当前会话状态，结构如下：

```json
{
  "version": "v2",
  "current_phase": 3,
  "current_task": 2,
  "checkpoint": {
    "phase": 3,
    "task": 2,
    "text": "Designer 生成 design-spec.md"
  },
  "pending_questions": [
    {
      "source": "copywriter",
      "file": "ai-office/questions/copywriter-questions.md",
      "questions": ["是否有品牌 slogan?", "Hero 图用真实照片还是插画?"]
    }
  ],
  "user_inputs": {
    "brand_slogan": "让世界更美好",
    "hero_image_type": "真实照片"
  },
  "outputs_status": {
    "copy.md": "completed_with_user_feedback",
    "design-spec.md": "in_progress",
    "index.html": "pending",
    "meta.md": "pending"
  }
}
```

### State Operations

```bash
# 先加载仓库内置 helper，避免手写 jq 表达式
source "$SKILL_ROOT/state-management.sh"

# 缺少 state.json 时才初始化，保留 --resume 现场
ensure_state_initialized

# 读取当前进度
CURRENT_PHASE="$(get_current_phase)"
CURRENT_TASK="$(get_current_task)"

# 写入简单字段时显式传 value_type
write_state "current_phase" "3" "number"
write_state "current_task" "1" "number"

# 对带点号/连字符的真实输出键，使用字面键 helper
mark_task_waiting_for_user "design-spec.md"
mark_task_completed "copy.md"

# 保存用户补充输入
save_user_input "hero_image_type" "真实照片"

# 添加待处理问题与 checkpoint，不要直接手拼 pending_questions JSON
add_pending_question "designer" "需要确认配色方案"
create_checkpoint 3 2 "Designer 等待用户确认配色方案"
```

### Recovery Logic

每个阶段开始时检查 state.json：
```bash
if [[ -f "ai-office/state.json" ]]; then
  source "$SKILL_ROOT/state-management.sh"
  CURRENT_PHASE=$(get_current_phase)
  log "从 Phase $CURRENT_PHASE 恢复..."
else
  CURRENT_PHASE=0
fi
```

---

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aimonj0729-ai/ai-office-landing-skill](https://github.com/aimonj0729-ai/ai-office-landing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
