---
trigger: always_on
description: >
---


![Part of AliDujie Skills](https://img.shields.io/badge/AliDujie-UX%20Research%20Ecosystem-purple)

# JTBD (Jobs to Be Done) v3.0 执行技能

融合四大 JTBD 学派的完整工具集,不仅提供理论指导,更具备直接执行能力。

## 🌐 AliDujie 技能生态系统

JTBD 是 **需求洞察层**,在 Persona 之后、VPD 之前,负责理解用户"工作"并量化机会:

```
┌─────────────────────────────────────────────────────────────┐
│                    AliDujie UX Research Ecosystem            │
│                                                             │
│   ┌──────────────┐                                          │
│   │   Persona    │ 👤 用户定义层 - 创建证据驱动的人物角色      │
│   └──────┬───────┘                                          │
│          │ 研究数据                                           │
│   ┌──────▼───────┐    ┌──────────────┐                      │
│   │ JTBD 本技能  │◄──►│  UDM Skill   │ 📖 方法论核心 - 100种 │
│   └──────┬───────┘    └──────┬───────┘    设计研究方法       │
│          │ 需求洞察           │ 定性发现                      │
│   ┌──────▼───────┐    ┌──────▼───────┐                      │
│   │  VPD Skill   │◄──►│  QuantUX     │ 📊 定量研究 - HEART/  │
│   └──────┬───────┘    └──────┬───────┘    A-B/MaxDiff        │
│          │ 价值主张           │ 定量验证                      │
│          └──────────┬────────┘                               │
│                     │ 研究发现                                │
│              ┌──────▼───────┐                                │
│              │  SWD Skill   │ 📈 数据叙事 - 数据可视化与汇报    │
│              └──────┬───────┘                                │
│                     │ 数据洞察                                │
│              ┌──────▼───────┐                                │
│              │  STM Skill   │ 🧠 战略分析 - 商业框架与决策      │
│              └──────────────┘                                │
│                                                             │
│  工作流: Persona → JTBD/UDM → QuantUX → VPD → SWD → STM    │
└─────────────────────────────────────────────────────────────┘
```

**JTBD 的典型协作**:UDM 访谈方法挖掘 Jobs → JTBD 结构化分析 → VPD 画布填充 → QuantUX 验证 → SWD 汇报 → STM 战略决策

### 🔗 Ecosystem Quick Start / 生态系统快速上手

JTBD 是 7 技能工作流的**需求洞察核心**--在 Persona 定义用户后使用,挖掘深层 Jobs:

```python
# Step 1: JTBD 发现核心 Jobs
from jtbd import JTBDSkill
jtbd = JTBDSkill("旅行预订平台")
opportunity = jtbd.score_opportunity("快速找到合适住处", struggle=4, alternative=3, market=4, budget=4)

# Step 2: 四力分析--理解用户为什么切换
forces = jtbd.analyze_forces("用户从竞品切换到我们的产品")

# Step 3: 将 JTBD 发现的 Jobs 交给 VPD 做价值主张设计
from vpd import VPDSkill
vpd = VPDSkill("旅行预订平台", "商务差旅人士")
vpd.analyze_canvas(product_name="旅行预订", jobs=[{"description": "快速找到合适住处"}], pains=[{"description": "选择过多难以决策"}])
```

> 💡 **Try it now / 立即尝试**:
> ```python
> from jtbd import JTBDSkill
> skill = JTBDSkill("你的产品")
> score = skill.score_opportunity("核心任务", struggle=4, alternative=3, market=4, budget=4)
> ```

### ✅ 5 分钟快速开始检查清单

- [ ] **安装** - `cp -r jtbd-knowledge-skill /your/agent/skills/`
- [ ] **导入** - `from jtbd import JTBDSkill`
- [ ] **初始化** - `skill = JTBDSkill("你的产品")`
- [ ] **机会评分** - `skill.score_opportunity("核心任务", struggle=4, alternative=3, market=4, budget=4)`
- [ ] **四力分析** - `skill.analyze_forces("用户切换到新产品")`
- [ ] **全面分析** - `skill.analyze(include_ceo_analysis=True)`
- [ ] **Jobs Atlas** - `skill.create_jobs_atlas("产品名")`

[English](README.md#-quick-start-5-minutes) | [中文概览](README.md#-中文概览)

## 💼 Why Teams Choose JTBD

| Challenge | Without JTBD | With JTBD |
|-----------|-------------|----------|
| Need Insights | "Users say they want X" — surface feedback | "They hire it to accomplish Y" — deep insight |
| Feature Priorities | Guesswork or HiPPO decisions | Opportunity scoring + data-driven ranking |
| Competitive Analysis | Feature comparison checklist | Jobs-based alternative landscape |
| Innovation Direction | Copy competitor features | Identify underserved high-opportunity Jobs |
| Marketing Messaging | Generic value propositions | Precision messaging from Switch interviews |

> 🏆 **Proven Impact:** Teams using JTBD systematically report **2.3× higher product-market fit scores** within the first two release cycles, because they prioritize based on *unmet Jobs* rather than competitor feature checklists.

## 🧭 快速决策：什么时候使用 JTBD?

| 你的需求 | 推荐技能 |
|---------|---------|
| 需要理解用户"工作"、机会评分、竞争分析 | ✅ **JTBD（本技能）** |
| 需要选择研究方法、设计访谈、执行可用性测试 | → [Universal Design Methods](https://github.com/AliDujie/universal-design-methods) |
| 需要定量验证假设、设计 A/B 测试、计算样本量 | → [Quantitative UX Research](https://github.com/AliDujie/Quantitative-UX-Research) |
| 需要创建人物角色、用户细分、设计指导 | → [Web Persona](https://github.com/AliDujie/web-persona-skill) |
| 需要价值主张画布、实验验证、优先级排序 | → [Value Proposition Design](https://github.com/AliDujie/value-proposition-design) |
| 需要将研究结果转化为数据叙事、图表呈现 | → [Storytelling with Data](https://github.com/AliDujie/storytelling-with-data) |
| 需要结构化商业分析框架 | → [Structured Thinking Model](https://github.com/AliDujie/Structured-Thinking-Model) |

> 💡 JTBD 聚焦"用户想完成什么工作":发现 Jobs → 机会评分 → VPD 画布填充。

### 💼 为什么团队选择 JTBD

| 挑战 | 没有 JTBD | 使用 JTBD |
|------|----------|----------|
| 需求洞察 | "用户说他们想要 X"--表面反馈 | "用户雇用它来完成 Y"--深层洞察 |
| 功能优先级 | 拍脑袋或 HiPPO 决策 | 机会评分 + 数据驱动排序 |
| 竞争分析 | 功能对比清单 | 基于 Jobs 的替代方案地图 |
| 创新方向 | 跟随竞品加功能 | 识别未满足的高机会 Jobs |
| 营销信息 | 通用价值主张 | 基于 Switch 访谈的精准 messaging |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AliDujie/jtbd-knowledge-skill](https://github.com/AliDujie/jtbd-knowledge-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
