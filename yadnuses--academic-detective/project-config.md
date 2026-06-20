---
trigger: always_on
description: Academic integrity investigation and scholar profile analysis system. Use when conducting comprehensive academic background checks, credential verification, publication analysis, or institutional affiliation audits of scholars, researchers, or faculty members. Triggers on phrases like "调查学者", "学术调查", "查某人的学术背景", "核实论文", "学术档案调查", "学者背调", or when analyzing academic misconduct, credential inflation, publication quality, or research integrity issues.
---


> ⚠️ **LLM 强制指令（请逐条执行）**：
> 
> 1. **优先使用现有脚本**。本系统的工具链位于 `scripts/` 目录下，一般情况下不要自己写 Python 代码替代现有工具。
> 2. **所有脚本路径以下方工具索引表为准**。`scripts/` 根目录下绝大多数 <500 字节的 `.py` 文件是兼容性 shim，真实实现位于子目录中。
> 3. **执行流程**：先 ls 确认文件存在 → 再执行。如果不确定某个脚本的具体路径或参数，先 Read 该脚本头部的 docstring。
> 4. **遵循七步顺序**：Basic Profile → Output Quantity → Quality Assessment → Relationship Network → Anomaly Detection → Multi-Source Validation → Report Generation。不要跳步。
> 5. **分层加载**：执行具体步骤前，先 Read 对应的 `docs/skill/` 文档获取详细流程。
> 6. **自由边界**：对于客户要求但没有在目录写出的功能，可由LLM编写脚本或自主筛查以保证功能实现。

# Academic Investigation Skill

Comprehensive academic background investigation system based on proven methodology from verified case studies.

## Overview

A systematic 7-step framework for investigating academic profiles, verifying credentials, analyzing publication quality, and identifying potential academic misconduct or credential inflation.

## When to Use

- Investigating a scholar's academic background or credentials
- Verifying publication claims or academic achievements
- Analyzing research quality and originality
- Checking for academic misconduct (plagiarism, duplicate publication, credential fraud)
- Auditing institutional affiliations and career progression
- Evaluating relationship networks and resource dependencies

## Semi-Automatic Workflow Philosophy

**Human-in-the-loop** design. Scripts assist with computation; human judgment drives decisions.

| Actor | Responsibilities |
|:------|:----------------|
| **Human** | CNKI/Wanfang/WoS searches, institutional verification, monograph acquisition, final interpretation |
| **LLM** | Tool selection, signal interpretation, hypothesis generation, report drafting |
| **Scripts** | Data normalization, quantitative analysis, text profiling, report assembly |

```
scripts/
├── core/                    # 共享基础设施
├── domestic/                # 国内学者调查适配器
├── international/           # 国际学者/导师调查适配器
├── cross_border/            # 海归学者调查（合并国内+国际）
├── analysis/                # 共享分析模块
├── network/                 # 关系网络与腐败图谱
├── deep_evidence/           # 深度证据层（数据取证/发表链/伦理/同行评议/证据编译）
├── report/                  # 报告生成
├── agents/                  # 多智能体协作层
├── delivery/                # 交付层（自检+格式化）
├── backend/                 # Web 后端 API
└── investigate.py           # CLI 编排入口
```

---

## 精准工具索引表

> 每个工具的真实路径已排除根目录兼容性 shim。执行前先 `ls` 确认文件存在。

### Core（基础设施）

| 工具 | 路径 | 功能 | 输入 | 输出 | 时机 | Track |
|:-----|:-----|:-----|:-----|:-----|:-----|:------|
| case_manager | `scripts/core/case_manager.py` | 案件注册与 ID 生成 (AD-YYYY-MM-DD-NNN) | 甲方名称 | case_id + 目录结构 | Step 0 | all |
| db | `scripts/core/db.py` | SQLite 案件数据库（9表） | case_id | .db 文件 | Step 0 后自动 | all |
| config_loader | `scripts/core/config_loader.py` | 统一配置加载（v1→v2 迁移） | config.yaml | config dict | 全程 | all |
| router | `scripts/core/router.py` | 调查类型路由 (domestic/international/cross_border) | config | track 判定 | Step 0 | all |
| watermark | `scripts/core/watermark.py` | 零宽水印嵌入/提取 | Markdown 报告 | 水印报告 | Step 8 交付 | all |
| utils | `scripts/core/utils.py` | 日志、JSON 存储等公共工具 | — | — | 全程 | all |

### Data Import（数据导入）

| 工具 | 路径 | 功能 | 输入 | 输出 | 时机 | Track | 依赖 |
|:-----|:-----|:-----|:-----|:-----|:-----|:------|:-----|
| data_importer | `scripts/domestic/data_importer.py` | CNKI/万方/WoS 导入 + 去重 | 导出的 txt/csv 文件 | 统一 JSON 论文列表 | Step 2 | domestic | init 完成 |
| data_fetcher | `scripts/international/data_fetcher.py` | OpenAlex/ORCID/S2/GS/PubPeer/RW/arXiv 自动抓取 | config.yaml (scholar name + IDs) | `auto_fetched.json` | Step 2 | international | init 完成 |
| openalex_enricher | `scripts/domestic/openalex_enricher.py` | 用 OpenAlex 补充国内论文的国际引用数据 | 论文 JSON | 增强后 JSON | Step 2 后 | domestic | data_importer |
| xiaohongshu_client | `scripts/international/xiaohongshu_client.py` | 小红书学生评价抓取 + 情感/维度提取 | 导师名/学校名 | 评价 JSON | Step 6 | international/cross_border | — |
| wechat_search | `scripts/domestic/wechat_search.py` | 微信公众号文章搜索（补充线索） | 关键词 | 文章列表 | Step 6 | domestic | — |
| review_matcher | `scripts/domestic/review_matcher.py` | 研学网评价表匹配 → investigation_leads | 导师名/学校 | 结构化 leads JSON | Step 6 | domestic | `_private/研学网导师评价表.xlsx` |

### Analysis（分析）

| 工具 | 路径 | 功能 | 输入 | 输出 | 时机 | Track | 依赖 |
|:-----|:-----|:-----|:-----|:-----|:-----|:------|:-----|
| text_profiler | `scripts/analysis/text_profiler.py` | PDF/Markdown/文本分析: 词频、原创性标记、引用模式、论文类型分类 | PDF/Markdown 文件 | JSON profile | Step 3 | all | 论文 PDF 获取 |
| paper_quality_rubric | `scripts/analysis/paper_quality_rubric.py` | 论文六维评分（创新性/方法/论证/文献/写作/贡献） | text_profiler 输出 JSON | 六维评分 JSON | Step 3 | all | text_profiler |
| hybrid_scorer | `scripts/analysis/hybrid_scorer.py` | 综合评分（合并多维度） | 多个评分 JSON | 综合分 JSON | Step 3 | all | paper_quality_rubric |
| stylometry_profiler | `scripts/analysis/stylometry_profiler.py` | 风格计量学：虚词频率、句法结构、相似度矩阵 | 多篇文本 | 相似度热力图 + JSON | Step 5 代笔检测 | all | 多篇 PDF |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yadnuses/Academic-Detective](https://github.com/yadnuses/Academic-Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
