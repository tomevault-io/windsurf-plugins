---
trigger: always_on
description: | 工作流 | 用途 | 触发方式 | 智能体数量 |
---

# CLAUDE.md - 多智能体协作配置

## 三个工作流

| 工作流 | 用途 | 触发方式 | 智能体数量 |
|--------|------|----------|------------|
| **编码工作流** | 通用软件开发 | "帮我写一个XXX功能" | 3个 |
| **PPT工作流** | 专业PPT制作 | "帮我做一个PPT" | 9个 |
| **论文工作流** | 学术论文/数学建模 | "帮我写一篇论文" | 8-11个（按类型） |

---

## 工作流1: 编码工作流

### 智能体角色

| 角色 | 职责 | 配置文件 |
|------|------|----------|
| 主智能体 (Orchestrator) | 任务协调、拆解、分配 | `orchestrator.md` |
| 编码智能体 (Coder) | 编写代码 | `coder.md` |
| 评审智能体 (Reviewer) | 评审代码质量 | `reviewer.md` |

### 工作流程
```
1. 主智能体接收任务
2. 拆解为子任务
3. 对每个子任务：编码(新)→评审(新)→不通过则修复重审(最多3轮)
4. 整合结果
```

### 调用方式
```
帮我写一个 [功能描述]
```

---

## 工作流2: PPT工作流

### 智能体角色（9个）

| 阶段 | 角色 | 配置文件 |
|------|------|----------|
| 0 | 素材提取师 | `ppt_extractor.md` |
| 1 | 需求分析师 | `ppt_requirement_analyst.md` |
| 2 | 内容研究员 | `ppt_content_researcher.md` |
| 3 | 架构设计师 | `ppt_framework_designer.md` |
| 4 | 协调智能体 | `ppt_coordinator.md` |
| 4a | 内容智能体(每页) | `ppt_page_content.md` |
| 4b | 素材智能体(每页) | `ppt_page_asset.md` |
| 4c | 审核智能体(每页) | `ppt_page_review.md` |
| 5 | 终审智能体 | `ppt_reviewer.md` |
| 6 | 交付协调员 | `ppt_delivery.md` |

### 工作流程
```
阶段0: 素材提取（可选）
阶段1: 需求确认 → requirements.md
阶段2: 内容搜索 → research/
阶段3: 逐页大纲 → outline.md
阶段4: 逐页协作构建（每页3-5分钟）：
  对每一页:
    内容智能体(新) → 布局规范
    素材智能体(新) → 页面代码
    审核智能体(新) → 审核报告（不通过则重试最多2次）
  协调智能体组装 → create_ppt.py + presentation.pptx + presentation.html
阶段5: 终审 → final_review.md
阶段6: 交付 → PPT_Delivery_*/
```

### ⛔ 素材库验证（阶段5.5 - 强制执行）

在调用终审智能体之前，检查生成的脚本是否使用了PPTBuilder：

```bash
grep -c "from ppt_builder import PPTBuilder\|PPTBuilder" [脚本文件]
```

匹配数为0则重新调用ppt_coder重写。最多重试2次。

### 设计规范
- 配色方案：学术蓝/学术绿/简约黑/暗色科技/农业绿/芯动忆生蓝/茶园绿
- 字体规范：微软雅黑 + Arial/Helvetica
- 图表规范：300 DPI+，PNG格式

### 时长模板

| 时长 | 页数 | 预计制作时间 |
|------|------|--------------|
| 5分钟 | 5-8页 | 1-1.5小时 |
| 15分钟 | 15-20页 | 2-3小时 |
| 30分钟 | 20-30页 | 4-5小时 |
| 60分钟 | 30-50页 | 6-8小时 |

---

## 工作流3: 论文工作流

### 快速使用
```
帮我写一篇论文，主题是：[你的主题]
```

### 论文类型分支

| 论文类型 | 触发关键词 | 架构设计师 | 编写智能体 |
|----------|------------|------------|------------|
| **通用学术论文** | 实证研究/综述/理论分析/毕业设计 | `paper_framework_designer.md` | `paper_content_writer.md` |
| **计算机/软件工程** | 计算机/软件/系统/算法/深度学习 | `paper_cs_framework_designer.md` | `paper_cs_content_writer.md` |
| **历史学** | 历史/史料/考证/古代/近代 | `paper_history_framework_designer.md` | `paper_history_content_writer.md` |
| **数学建模** | 数学建模/建模竞赛/国赛/美赛 | `paper_math_modeling_framework_designer.md` | `paper_math_modeling_content_writer.md` |

### 智能体角色

#### 通用智能体（所有类型共享）

| 阶段 | 角色 | 配置文件 |
|------|------|----------|
| 1 | 需求分析师 | `paper_requirement_analyst.md` |
| 1.5 | 代码阅读智能体 | `paper_code_reader.md` |
| 2 | 文献搜索智能体 | `paper_literature_searcher.md` |
| 3 | 文献综述智能体 | `paper_literature_reviewer.md` |
| 4.5 | ⚠️ 结果构建智能体(仅数学建模) | `paper_math_modeling_result_builder.md` |
| 5.5 | 格式检查智能体 | `paper_format_checker.md` |
| 6 | 学术润色智能体 | `paper_polishing.md` |
| 8.5 | ⚠️ 一致性审计智能体(仅数学建模) | `paper_consistency_auditor.md` |
| 8.6 | ⚠️ 图表账本检查(自动化脚本) | `artifact_checker.py` |
| 8.7 | ⚠️ 正文验证智能体(仅数学建模) | `paper_final_consistency_verifier.md` |
| 7 | 审稿回复智能体 | `paper_review_response.md` |
| 8 | PPT生成智能体 | `paper_ppt_generator.md` |
| 9 | 终审交付智能体 | `paper_delivery.md` |

### 工作流程

```
【通用学术/计算机/历史分支】          【数学建模分支（13阶段）】
        ↓                                   ↓
阶段1: 需求分析                      阶段1: problem_parser → problem_spec.json
        ↓                                   ↓
阶段1.5: 代码阅读(可选)              阶段2: solution_planner → solution_plan.json
        ↓                                   ↓
阶段2: 文献搜索                      阶段3: result_builder → results_master.json
        ↓                                   ↓
阶段3: 文献综述                      阶段4: canonical_table_gen → tables/*.csv
        ↓                                   ↓
阶段4: 结构设计                      阶段5: figure_generator → figures/*.png
        ↓                                   ↓
阶段5: 逐章写作                      阶段6: artifact_manifest_builder → artifact_manifest.json
        ↓                                   ↓
阶段6-9: 润色→审稿→交付              阶段7: chapter_writer → chapters/chapter_N.docx（骨架模式）
                                       ↓
                                      阶段8: evidence_based_expander → chapter_N_expanded.docx
                                       ↓
                                      阶段9: abstract_writer → 摘要/结论（最后编写）
                                       ↓
                                      阶段10: consistency_checker → 结果账本内部一致性
                                       ↓
                                      阶段11: artifact_checker → 图表账本一致性
                                       ↓
                                      阶段12: document_verifier → 正文与账本一致性
                                       ↓
                                      阶段13: delivery → 终审交付
```

#### 逐章写作循环（阶段7→8）
```
对每章:
  chapter_writer(新) → chapter_{N}.docx + chapter_{N}_summary.md + chapter_{N}_requests.md
  evidence_based_expander(新) → chapter_{N}_expanded.docx + chapter_{N}_expansion_log.md
```

#### 检查阶段 fail-fast
```
阶段10-12 任一 checker status=failed → 立即停止，输出错误报告
不得跳过检查进入交付阶段
```

#### 回退通道
- 阶段7→阶段4：大纲逻辑不通 → 输出 rollback_feedback.md → 重新设计大纲
- 阶段7→阶段2：文献不足 → 输出 rollback_feedback.md → 补搜文献
- 回退次数限制：同一方向最多1次，累计最多2次

### 需求确认时询问的问题

| 问题 | 选项 | 默认值 |
|------|------|--------|
| 学历层次 | 课堂作业 / 专科 / 本科 / 硕士 / 博士 | 本科 |
| 论文类型 | 实证研究 / 综述 / 理论分析 / 毕业设计 | 实证研究 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LiJzd/paper-agent-pipeline](https://github.com/LiJzd/paper-agent-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
