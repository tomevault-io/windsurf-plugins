---
trigger: always_on
description: >
---


# Google Ads B2B 竞争分析与投放策略专家

## 概述

本技能使 Agent 能够作为 Google Ads 投放专家和 SEO 分析师，基于用户提供的网站 URL，全面梳理目标网站的 Google Ads 广告投放策略。核心能力包括：网站 SEO 架构分析、竞对关键词策略挖掘（逐家深度分析）、B2B 关键词体系构建、广告定位策略推导、完整投放执行路线图。

## 触发条件

当用户提出以下类型请求时使用本技能：
- "分析 [网站URL] 的 Google Ads 投放策略"
- "[网站URL] 的广告关键词策略是什么"
- "帮我梳理竞对网站的谷歌广告投放策略"
- "分析 [网站URL]，提取其 SEO 和广告投放关键词"
- "基于 [网站URL] 制定 Google Ads 投放计划"
- 任何涉及基于网站 URL 分析竞对广告策略的场景

**⚠️ 如果用户未提供 URL**：先执行「前置检查」中的 URL 有效性检查 → 回复「请提供目标网站完整 URL（如 https://example.com）」，**不得尝试猜测或虚构 URL**。

---

## 前置检查与失败处理

| 检查项 | 通过条件 | 失败触发条件 | Fallback 动作 |
|--------|---------|-------------|--------------|
| URL 有效性 | 用户提供有效 URL | 用户未提供 URL / URL 格式错误 / 网站 404/403 | 🔴 STOP：礼貌回复「请提供目标网站完整 URL（如 https://example.com），我将为您分析其 Google Ads 投放策略」 |
| 网站可访问性 | browser_visit 成功返回页面 | 超时 / 屏蔽爬虫 / Cloudflare 拦截 | 尝试 web_search `cache:{URL}` 或 `site:{domain}` 获取缓存快照；仍失败 → 标注「网站无法直接访问，以下分析基于公开索引数据」 |
| 竞对数量 | 识别到 ≥5 家核心竞对 | 仅识别到 <5 家 | 扩大搜索词范围（从品牌词扩展到品类词 + 行业词）；仍不足 → 标注「竞对数据有限，建议补充行业报告」并继续分析 |
| B2B 属性确认 | 网站明确为 B2B 或 B2B+B2C 混合 | 纯 B2C 电商 / 内容站 / 个人博客 | 🔴 STOP：回复「本技能专注于 B2B 广告投放策略分析。该网站看起来是 B2C/内容型站点，建议转用 B2C 广告分析工具」 |
| 地理范围 | 明确目标市场（US/EU/China/其他） | 网站无国家/语言标识 / 用户未指定 | 🔴 CHECKPOINT：询问用户「该网站主要目标市场是哪个地区？（美国/欧洲/中国/全球）」——不同地区的关键词策略、出价水平、竞对格局差异显著 |

---

## 分析流程

执行分析时严格遵循以下 6 个步骤，每个步骤完成后进入下一步：

### 步骤 1：网站基础分析（定量画像）

使用 browser_visit 工具访问目标网站，记录：
- **定量企业数据**：成立年份、公司规模、年收入/融资额、客户数量、市场份额
- **业务核心信息**：产品/服务矩阵（至少列出 5-10 个核心产品）、目标行业（列出 5-10 个）、目标客户类型
- **页面 SEO 元素**：Title、Meta Description、H1-H3 标签、结构化数据类型
- **技术 SEO 特征**：加载速度、移动端体验、URL 结构、内部链接策略

### 步骤 2：SEO 关键词深度提取

提取页面现有关键词布局（参考 `references/seo-analysis-guide.md` —— 含页面元素抓取清单、关键词密度计算方式、语义分析工具推荐）：
- 页面源码中所有 meta keywords、语义关键词
- H1-H6 标题标签中的关键词分布
- 内容主题聚焦分析（识别 3-5 个核心主题方向）
- 图片 alt 文本中的关键词
- 内部链接锚文本分析

### 步骤 3：竞对识别与逐家深度分析

**3.1 竞对识别**（使用 web_search，目标：找到 5-8 家核心竞对）

| 搜索组合 | 用途 |
|---------|------|
| `{品牌名} vs` / `{品牌名} alternative` | 发现直接竞对 |
| `{核心产品} best/top {品类}` | 发现行业主要玩家 |
| `{品类} supplier/manufacturer` | 发现 B2B 供应链竞对 |

→ 输出：竞对候选清单（≥5 家），含域名 + 一句话定位

**3.2 逐家竞对深度分析**（对每家执行以下 5 项检查）

对竞对候选清单中的每家，按顺序检查：

| 检查项 | 工具 | 输出 |
|--------|------|------|
| ① 基础画像 | browser_visit | 定位、核心产品、目标市场 |
| ② SEO 特征 | browser_visit | Title、H1、核心关键词部署 |
| ③ 广告痕迹 | web_search `{核心词}` | 是否出现竞对广告、文案风格 |
| ④ 内容策略 | browser_visit `/blog` 或 `/resources` | 博客主题、案例研究、白皮书方向 |
| ⑤ 差异化定位 | 对比分析 | 与目标网站的 3 个关键差异 |

→ 输出：每家竞对 1 张分析卡片（按「输出模板」3.2 节格式）

**3.3 竞对策略共性总结**
- 对比 ≥5 家竞对的数据，提炼 3-5 条行业共性规律
- 标注：哪些是行业通用策略 vs 哪家有独特打法

### 步骤 4：B2B 关键词体系构建

按搜索意图分类：信息型、商业调研型、交易型、导航型
按漏斗阶段分类：认知阶段、考虑阶段、决策阶段、忠诚阶段
按关键词类型分类：行业词、产品词、品牌词、竞品词、长尾词

**参考 `references/b2b-keyword-methodology.md` —— 含 B2B 关键词分类矩阵、意图映射表、长尾词挖掘方法、搜索量估算基准。**

### 步骤 5：否定关键词与数据闭环策略

**参考 `references/google-ads-framework.md` —— 含 Google Ads 账户结构最佳实践、出价策略选择矩阵、B2B 转化跟踪配置指南。**

**5.1 否定关键词体系**
- 通用否定词：free, cheap, DIY, jobs, career, resume, hiring, tutorial, Wikipedia
- B2B 特定否定词：retail, consumer, personal, home use, individual
- 地理不匹配否定词：排除不服务的国家/地区
- 业务不匹配否定词：排除不生产的产品规格/型号

**5.2 CRM 数据闭环（B2B 必做）**
- GCLID 捕获与传递机制
- 线下转化导入（Offline Conversion Import）
- Enhanced Conversions for Leads
- 线索评分回传优化

### 步骤 6：广告策略综合报告

🔴 **CHECKPOINT**：生成完整报告前，先向用户确认：
> 「分析完成，即将生成包含 12 个章节的完整报告 + 3 个 CSV 文件。报告预计 600-900 行。是否按此输出？或您希望重点关注某几个章节（如只输出关键词分析 + CSV）？」

- 用户确认 → 按完整模板输出
- 用户要求精简 → 只输出指定章节，但 CSV 文件仍必须生成
- 用户要求扩展 → 在对应章节追加深度分析

整合所有分析结果，按输出模板生成报告。**报告篇幅目标：600-900 行 Markdown，确保内容深度与 baseline 一致。**

## CSV 导出功能

在生成完整的策略分析报告后，**必须**同时输出可导入 Google Ads Editor 的 CSV 文件，存储到 `/Users/steven/workspace/fbkpower-seo/06-deliverables/` 目录。

**参考 `references/csv-export-guide.md` —— 含 Google Ads Editor 导入字段映射表、UTF-8 BOM 编码处理、常见导入错误排查。**

### CSV 文件规范

**文件 1: 关键词与Campaign结构 CSV**
- 文件名格式: `google-ads-{domain}-campaign-{date}.csv`
- 必须包含的列: Campaign, Campaign Type, Campaign Status, Budget, Ad Group, Max CPC, Keyword, Criterion Type, Status, Final URL
- 关键词必须标注匹配类型: Exact / Phrase / Broad / Negative Broad / Negative Phrase / Negative Exact
- 否定关键词单独列出，Criterion Type 为 Negative Broad

**文件 2: RSA 广告文案 CSV**
- 文件名格式: `google-ads-{domain}-rsa-{date}.csv`
- 必须包含的列: Campaign, Ad Group, Ad Type, Headline 1-15, Description 1-4, Path 1, Path 2, Final URL
- 每个 Ad Group 至少 2 套 RSA

**文件 3: 否定关键词清单 CSV**
- 文件名格式: `google-ads-{domain}-negatives-{date}.csv`
- 必须包含的列: Campaign, Ad Group, Negative Keyword, Match Type, Reason
- 按类别分组: 通用否定词 / B2B特定否定词 / 业务特定否定词 / 地理排除词

### CSV 生成要求
- 使用 `execute_code` 工具生成 CSV 文件
- 文件编码: UTF-8 with BOM (Excel兼容)
- 字段分隔符: 逗号
- 文本限定符: 双引号
- 生成后向用户报告文件路径和总行数

## 输出模板

ALWAYS 使用以下完整结构输出分析报告（如某个章节确实无数据，标注"未检测到"而非省略），并在报告末尾附加 CSV 文件交付清单：

```markdown
# [目标网站域名] Google Ads 投放策略分析报告

## 一、网站基础画像
### 1.1 企业核心数据
| 项目 | 内容 |
|------|------|
| 网站域名 | [domain] |
| 成立年份 | [年份] |
| 公司规模 | [员工数] |
| 年收入/融资 | [金额] |
| 客户数量 | [数量/市场份额] |
| 业务类型 | [B2B/B2C/混合] |
| 页面语言/市场 | [语言/地区] |

### 1.2 核心产品/服务矩阵
| 产品/服务 | 描述 | 目标客群 |
|-----------|------|----------|
| [产品1] | [描述] | [客群] |
| [产品2] | [描述] | [客群] |

### 1.3 目标行业与客群
- 核心行业：[行业1]、[行业2]、[行业3]...
- 决策角色：[角色1]、[角色2]、[角色3]...
- 客单价区间：[金额范围]

### 1.4 技术 SEO 特征
- Title: [内容]
- Meta Description: [内容]
- H1: [内容]
- 结构化数据: [有/无/类型]
- 加载速度: [快/中/慢]

---

## 二、SEO 关键词现状
### 2.1 页面已部署关键词
| 关键词 | 位置（Title/H1/Content） | 密度 | 搜索意图 |
|--------|-------------------------|------|----------|
| [词1] | [位置] | [密度] | [意图] |
| [词2] | [位置] | [密度] | [意图] |

### 2.2 内容主题聚焦
[描述网站内容的 3-5 个主要主题方向，每个方向列出核心关键词]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dongtonghui/google-ads-strategist](https://github.com/dongtonghui/google-ads-strategist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
