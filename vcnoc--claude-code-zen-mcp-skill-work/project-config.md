---
trigger: always_on
description: - <!-- bootstrap: lang=zh-CN; encoding=UTF-8 -->
---

- <!-- bootstrap: lang=zh-CN; encoding=UTF-8 -->
  <!-- AGENTS_VERSION: 2025-10-11.4 (Phases-only routing, heading cleanup) -->

  # CLAUDE.md（Router → Phases）面向 AI 编程智能体的「轻量路由 + 多阶段 + 知识库驱动」规则集（项目配置文件）

  > 目的：基于全局规则和路由机制处理当前用户消息，结合 P1-P3 阶段规则进行响应（**P4 不参与初始路由**），参考`项目知识库内容结构与生成规则统一模板`生成正确的项目知识库文档。

  ---

  ## 📚 共享概念速查(SSOT)

  > 以下概念在整个规则体系中只此一处定义,所有技能文档必须引用而非重复。

  ### automation_mode (自动化模式)

  **核心理念：全自动模式 = 零等待原则（Zero-Wait Principle）**

  automation_mode=true 的本质含义：
  - 用户说"全程自动化" = 用户已将**所有决策权**委托给 AI
  - 除阻塞性错误（环境缺失、依赖错误、安全风险）外，**禁止在任何节点等待用户确认**
  - "零等待"不是建议，是**强制约束** - 违反此原则 = 违背用户授权意图

  **记忆锚点**：
  - automation_mode=true → 像执行 shell 脚本一样自动运行到底
  - 技能返回 = 脚本执行到下一行，不是暂停点
  - 询问用户 = 违反原则（除非阻塞性错误）

  **技术规范**：
  - **谁设**: 仅 main-router 在任务开始时判断并设置
  - **触发关键词**: "全程自动化" / "自动化流程" / "全自动" / "自动化模式"
  - **取值**: `true`(全自动) / `false`(交互,默认)
  - **传递**: `[AUTOMATION_MODE: true]` (通过上下文)
  - **下游约束**: 只读取,不重新判断;true 时禁止询问用户"是否继续?"
  - **例外情况**: 阻塞性错误（环境缺失、依赖错误、权限问题）、安全风险（敏感信息暴露、生产环境操作）

  ### coverage_target (测试覆盖率目标)
  - **谁设**: 仅 main-router 在 P1/P2 阶段询问用户并设置
  - **默认**: 85% (工业标准) / **最低**: 70%
  - **传递**: `[COVERAGE_TARGET: 85%]` (通过上下文)
  - **下游约束**: 只读取,不询问用户;验证时使用该值作为标准
  - **行为规范**: 见 [G9 测试覆盖率目标设定] (包含 < 70% 强制报错等逻辑)

  ### auto_log (自动化决策日志)
  - **触发**: 仅当 automation_mode=true 时生成
  - **三层架构**:
    - **Layer 1 核心模板** (约200行): `skills/shared/auto_log_template.md` - 常驻内存,提供7个必选章节骨架
    - **Layer 2 详细规范** (约400行): `skills/shared/auto_log_detailed_spec.md` - 按需引用,包含信息提取规则、质量检查清单、FAQ
    - **Layer 3 示例库** (约300行): `skills/shared/auto_log_examples.md` - 仅参考时查阅,包含完整示例和决策树可视化
  - **职责分工**: 技能输出片段 → router 汇总 → simple-gemini 生成 auto_log.md
  - **文件性质**: 运行时审计日志,不纳入版本控制
  - **Layer选择标准**（自动判断）:
    - **仅Layer 1**：简单任务（耗时<30分钟 且 阶段≤3 且 无P4触发）
    - **Layer 1+2**：复杂任务（耗时≥30分钟 或 含P4阶段 或 质量问题≥5个）
    - **Layer 1+2+3**：需要参考示例时（首次生成 或 复杂决策树可视化需求）

  ### 复杂操作规范(独立文档)
  以下规范因操作步骤复杂,抽取为独立文件:
  - **G10 环境自适应 CLI 调用**: 见 `references/standards/cli_env_g10.md`
  - **P4 最终质量验证**: 见 `references/standards/p4_final_validation.md`

  ---

  ## 核心工作流（优先执行）

  **智能技能路由优先原则：**

  在处理**任何用户请求**之前，应优先使用 **main-router skill** 进行智能路由和技能匹配。main-router 将基于以下标准自动选择最合适的工具或技能：

  - **标准文件**：全局和项目级的 CLAUDE.md
  - **当前阶段**：P1 (分析问题) / P2 (制定方案) / P3 (执行方案) / P4 (错误处理)
  - **用户意图**：问答、深度分析、代码审查、文档生成、规划制定等

  **可用技能/工具：**
  - `zen-chat` - 一般问答和概念解释
  - `zen-thinkdeep` - 复杂问题深度调查
  - `codex-code-reviewer` - 代码质量审查（5 维度检查）**[代码完成后强制使用]**
  - `simple-gemini` - 标准文档和测试代码生成 **[文档/测试生成强制使用]**
  - `deep-gemini` - 深度技术分析文档（含复杂度分析）
  - `plan-down` - 智能规划与任务分解 **[plan.md 生成强制使用]**

  **职责分配（Responsibility Matrix）：**

  | 角色 | 职责 | 执行时机 | 调用方式 |
  |------|------|----------|----------|
  | **主模型**<br/>(Claude Code 主会话) | - 接收用户请求<br/>- 调用 main-router skill（任务开始时）<br/>- 调用其他技能（根据需要）<br/>- **执行自动恢复检查**（技能返回后）<br/>- 创建和更新 TodoList<br/>- 输出结果给用户 | 整个任务生命周期 | - |
  | **main-router skill** | - 意图识别<br/>- 阶段匹配<br/>- 设置 automation_mode<br/>- 选择最佳技能 | 任务开始时（一次性） | 主模型调用 → 返回建议 |
  | **plan-down skill** | - 方法清晰度判断<br/>- 任务分解<br/>- 生成 plan.md | 需要规划时（一次性） | 主模型调用 → 返回 plan.md |
  | **codex skill** | - 代码质量检查<br/>- 5 维度验证 | 代码完成后（一次性或多次） | 主模型调用 → 返回检查报告 |
  | **simple-gemini skill** | - 文档生成<br/>- 测试生成 | 需要文档/测试时（多次） | 主模型调用 → 返回文档/测试代码 |

  **关键点**：
  - 所有 skill 都是**一次性调用**，执行完**返回给主模型**
  - **主模型**负责整个流程的控制和自动恢复
  - main-router 负责初始路由和全程监控，在关键节点主动调用专用技能（遵循 G11 Anti-Lazy 原则）

  **职责分工（符合 G11）**：
  - **main-router**：初始路由 + 全程监控 + 强制调用技能（Anti-Lazy 原则）
  - **主模型**：执行任务 + 技能返回后自动恢复 + 遵循 router 监控指令
  - **技能（skills）**：一次性调用，返回给主模型，不持续运行

  **工作流程（含自动化模式状态管理）：**
  ```
  用户请求
    ↓
  主模型调用 main-router skill
    ↓
  main-router 读取标准文件 (CLAUDE.md)
    ↓
  CRITICAL:判断并设置 automation_mode 状态标志
    - 检测关键词："全程自动化" / "自动化流程" / "全自动" / "自动化模式"
    - 设置全局状态: automation_mode = true/false
    - 该状态在整个任务生命周期中保持不变
    ↓
  意图分析 + 阶段匹配 + 置信度评分
    ↓
  选择最佳技能/工具 (或直接执行)
    ↓
  main-router 返回建议给主模型
    ↓
  主模型执行任务 ← main-router 持续监控（Anti-Lazy），主模型负责自动恢复
    - 所有下游技能从上下文中读取 automation_mode
    - 禁止下游技能重新判断自动化模式
    - 技能返回后立即执行自动恢复检查
    ↓
  关键节点 main-router 强制调用技能（主模型执行）：
  - 代码完成 → 调用 codex skill（继承 automation_mode）
  - 需要测试 → 调用 simple-gemini skill → codex 验证（继承 automation_mode）
  - 需要规划 → 调用 plan-down skill（继承 automation_mode）
  - 需要文档 → 调用 simple-gemini/deep-gemini skill（继承 automation_mode）
  ```

  **强制技能使用规则（绝不偷懒）：**
  - **plan.md 必须用 plan-down**：禁止主模型直接写 plan.md
  - **代码完成必须用 codex**：任何代码生成/修改后都要质量检查
  - **测试生成工作流**：simple-gemini 生成 → codex 验证 → 主模型运行
  - **文档生成必须用专用技能**：不允许主模型直接生成正式文档

  **技能返回后的自动恢复（Skill Return Auto-Resume）**

  **执行主体：主模型** | **⚠️ CRITICAL - 技能返回立即执行，不可跳过**

  **【强制检查清单】** - 技能（plan-down, codex, simple-gemini等）返回后逐项执行：

  □ **Step 1**: 读取状态 - `[AUTOMATION_MODE: true/false]` + TodoList（pending/in_progress 数量）
  □ **Step 2**: 确认阶段 - P1/P2/P3/P4 + 刚才调用的技能
  □ **Step 3**: 判断推进
     - **阶段跃迁**：
       - P1完成 → 调用plan-down（P2）
       - P2完成 → 检查P3前置 → 满足则进P3
     - **阶段内推进**：
       - P3进行中 → 下一个pending任务
       - P4完成 → 回归闸门
  □ **Step 4**: 执行决策
     - automation_mode=true → **立即推进**（禁止询问）
     - automation_mode=false → 输出建议，等待确认

  **输出格式**：
  ```
  [全自动模式] 技能返回后自动推进
  ✅ 状态：automation_mode=true
  ✅ 阶段：P2完成 → P3前置条件通过
  ✅ 任务：检测到18个待执行任务
  → 自动进入P3阶段，开始执行...
  ```

  **❌ 禁止**：技能返回后结束对话 / 询问"是否继续" / 跳过检查 / 视为任务终点


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VCnoC) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
