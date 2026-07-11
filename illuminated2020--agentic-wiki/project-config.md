---
trigger: always_on
description: 本目录是一个由 Agent 维护、由人和 Agent 共同消费的**多域个人第二大脑**。所有知识库操作必须先理解本文件，再按相关域的 `CLAUDE.md` 执行。
---

# Agentic Wiki 操作协议

本目录是一个由 Agent 维护、由人和 Agent 共同消费的**多域个人第二大脑**。所有知识库操作必须先理解本文件，再按相关域的 `CLAUDE.md` 执行。

## 规则权威边界

- wiki skill 负责意图路由、操作编排和 Lint 的实际检查方式。
- 本文件负责项目边界、工作流契约、通用页面字段、命名、链接、索引、日志和分享边界。
- `sources/CLAUDE.md` 与各知识域的 `CLAUDE.md` 负责本域页面类型、局部字段、正文结构和专属 workflow。
- 每条规则只在其作用域内定义一次：全库不变量放根协议，域内数据契约跟随数据放在域协议中。

## 项目定位

- 定位：个人第二大脑——技术知识、工程经验、人际关系、想法捕获、写作管线、反思与原则。
- 基础设施层：`sources/` 保存原文和 source 摘要，`logs/` 保存追加式操作历史。
- 知识域：`tech/`、`entities/`、`qa/`、`learning/`、`craft/`、`ideas/`、`people/`、`writing/`、`reflections/`。
- 人侧界面：Obsidian。
- 目标：把资料加工成可链接、可追溯、可复用的长期知识，而不是堆积收藏。

## 渐进式加载（默认读取顺序）

所有 wiki 操作先读取共同入口，再按 Query 与写操作分流。

### 共同入口

1. `purpose.md`：确认项目边界。
2. 本文件（`CLAUDE.md`）：确认路由和全局规则。
3. `index.md`：全库内容导航。

### Query 分支

4. 相关域的 `index.md`。
5. 具体内容页。
6. 若发现需要写回，再进入写操作分支。

### 写操作分支（Ingest / Update / Lint / Restructure）

4. 相关域或基础设施层的 `CLAUDE.md`：确认局部页面契约和 workflow。
5. 相关域的 `index.md`。
6. 具体页面。

不要一开始扫描全库。域级 `CLAUDE.md` 是规则文件，不承担内容导航；内容导航一律走 `index.md`。

注意：这里限制的是**阅读理解和内容定位**，不是命令式结构验证。Lint、写后验证、断链检查、索引覆盖检查可以并且应该用 `find` / `grep` / `rg` 等命令扫描相关范围或全库。

## 域路由

AI 收到任务时，根据内容自行判断属于哪个域。执行前必须声明：

> "我判断这属于 X 域，读取 X/CLAUDE.md。"

如果任务跨域（例如从技术文章中提炼工程经验），声明主域和关联域：

> "这主要属于 craft/ 域，同时关联 tech/ 域。读取 craft/CLAUDE.md。"

### 域判断规则

| 信号 | 归属域 |
|------|--------|
| 技术文章入库、概念整理、来源分析、技术对比 | `tech/` |
| 组织、产品、协议、工具、框架、模型、数据集等具名非人类对象 | `entities/` |
| 对话中产生的有价值结论、讨论沉淀 | `qa/` |
| 学习路径规划、刷题记录、进度追踪 | `learning/` |
| 个人工程实践总结、方法论提炼、踩坑经验 | `craft/` |
| 产品灵感、项目构想、写作种子 | `ideas/` |
| 人物信息、互动记录、账号管理 | `people/` |
| 长文写作、文章状态跟踪 | `writing/` |
| 个人原则、认知变化、阶段性反思 | `reflections/` |

### 域边界歧义

如果内容同时符合两个或更多域，不得只按关键词或最先想到的域路由：

1. 列出候选主域和关联域。
2. 读取所有候选域的 `CLAUDE.md`；基础设施层候选同时读取其协议。
3. 按页面的主要职责判断归属：它长期要回答什么问题、由哪个 index 导航、后续按哪个 workflow 维护。
4. 搜索候选域的 index 和现有页面，检查同义、近义或已承担该职责的页面。
5. 声明主域、关联域，以及不选择其他候选域作为主域的理由。
6. 同一知识只保留一个 canonical 页面；跨域价值通过 Wikilink、`related` 或关联页面表达，不复制建页。

只有存在域歧义时才执行上述比较；明确内容继续按路由表和目标域协议渐进加载，不要求每次读取所有域。

## 目录边界

| 路径 | 规则 |
|------|------|
| `sources/` | 基础设施层。分为 `raw/`（不可变原文）和媒介类型子目录（AI 摘要页）。 |
| `sources/CLAUDE.md` | 来源层页面契约、raw/source 边界和项目快照 workflow。 |
| `sources/raw/` | **不可变原文存储**。抓取或粘贴的完整原始内容，LLM 只读不写（新建除外）。这是知识库的真理来源。 |
| `sources/articles/` | AI 生成的 source 摘要页（博客、公众号、新闻、essays），可随理解和来源协议演进更新。 |
| `sources/books/` | 书籍摘要页。 |
| `sources/papers/` | 学术论文、技术报告摘要页。 |
| `sources/videos/` | 视频来源摘要页。 |
| `sources/podcasts/` | 播客来源摘要页。 |
| `sources/notes/` | 个人笔记、对话记录、会议纪要。 |
| `sources/projects/` | 外部项目、teach 工作区、代码阅读材料的自包含快照。外部路径只能作为 provenance，不得作为唯一证据。 |
| `tech/` | 技术知识域。遵循 `tech/CLAUDE.md`。 |
| `tech/concepts/` | 概念、机制、方法论。 |
| `tech/queries/` | 未解决问题。 |
| `tech/comparisons/` | 横向对比。 |
| `tech/synthesis/` | 跨来源综合结论。 |
| `tech/index.md` | 页面导航入口，新增页面必须更新。 |
| `tech/overview.md` | 全库当前认知主线总览，知识库主线变化时更新。 |
| `entities/` | 具名非人类对象（组织、产品、协议、工具、框架、模型、数据集）。遵循 `entities/CLAUDE.md`。 |
| `entities/index.md` | 实体导航入口，新增实体必须更新。 |
| `logs/` | 基础设施层。操作日志按月归档（`logs/YYYY-MM.md`），严格只追加。 |
| `logs/index.md` | 月度日志导航入口。 |
| `index.md` | 全库内容导航唯一入口，只列知识域与基础设施 index、追加式列表、协议入口。 |
| `qa/` | 问答沉淀域。遵循 `qa/CLAUDE.md`。 |
| `learning/` | 学习路径域。遵循 `learning/CLAUDE.md`。 |
| `craft/` | 工程经验域。遵循 `craft/CLAUDE.md`。 |
| `craft/practices/` | 经验条目。 |
| `ideas/` | 想法捕获域。遵循 `ideas/CLAUDE.md`。**AI 不自动写入。** |
| `ideas/writing.md` | 写作类想法。 |
| `ideas/products.md` | 产品类想法。 |
| `people/` | 人际关系域。遵循 `people/CLAUDE.md`。 |
| `people/real/` | 真实关系人。 |
| `people/public/` | 公开人物。 |
| `people/accounts/` | 关注的内容账号。 |
| `writing/` | 写作管线域。遵循 `writing/CLAUDE.md`。 |
| `writing/ideas.md` | 写作主题池。 |
| `writing/articles/` | 文章条目。 |
| `reflections/` | 反思与原则域。遵循 `reflections/CLAUDE.md`。 |
| `reflections/principles.md` | 个人原则汇总。 |
| `todo.md` | 按域分类的任务列表。**AI 不自动写入，仅在用户确认后追加。** |

## 全局页面契约

本节定义所有域共享的不变量。具体页面类型的局部字段、正文结构和专属状态流转由其所在目录的 `CLAUDE.md` 定义。

### 页面类型路由

| Type | 权威协议 | 典型位置 |
|---|---|---|
| `source` | `sources/CLAUDE.md` | `sources/articles/`、`sources/papers/`、`sources/projects/` 等 |
| `entity` | `entities/CLAUDE.md` | `entities/` |
| `concept`、`query`、`comparison`、`synthesis`、`overview` | `tech/CLAUDE.md` | `tech/` |
| `qa` | `qa/CLAUDE.md` | `qa/` |
| `learning` | `learning/CLAUDE.md` | `learning/` |
| `practice` | `craft/CLAUDE.md` | `craft/practices/` |
| `idea` | `ideas/CLAUDE.md` | `ideas/writing.md`、`ideas/products.md` |
| `person` | `people/CLAUDE.md` | `people/real/`、`people/public/`、`people/accounts/` |
| `article` | `writing/CLAUDE.md` | `writing/articles/` |
| `reflection` | `reflections/CLAUDE.md` | `reflections/` |
| `index`、`log` | 本文件 | 根/域级 index、`logs/YYYY-MM.md` |

新增页面类型时，优先在归属域的 `CLAUDE.md` 定义；只有跨域不变量变化时才修改本节。

### 建页门槛

新建页面前必须先给出候选决策：

```markdown
| 候选 | 类型 | 决策 | 原因 | 目标页面 |
|---|---|---|---|---|
| 名称 | 页面类型 | Create/Update/Skip | 简短原因 | 页面名或现有链接 |
```

只有满足以下任一条件才允许新建独立页面：

- 会被多个页面反复引用。
- 支撑当前知识库主线，或改变已有判断。
- 对后续技术决策、写作、学习、关系维护或工程实践有长期价值。
- 需要长期追踪版本、关系、状态或未解决问题。

一次性提及、只有一句定义、与已有页同义或真实性未确认的名词默认留在 source、query 或已有页面中。

### 命名规范

- 文件名优先使用人类可读标题；中文概念允许中文文件名。
- 多词文件名使用空格，不使用连字符或下划线，例如 `Model Context Protocol.md`。
- 人物使用姓名或常用称呼；英文项目、协议、模型和工具保留官方名称。
- 同一对象只保留一个 canonical 页面，别名写入 `aliases`。
- 避免 `xxx 1.md`、`xxx_副本.md` 等临时命名；新建前先搜索相关域 index 和目录。

### 通用 Frontmatter

除 raw、月度日志和追加式列表外，普通页面继承以下字段：

```yaml
---
type: concept
title: 页面显示名
description: 一句话说明页面内容
aliases: []
tags: []
related: []

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Illuminated2020/agentic-wiki](https://github.com/Illuminated2020/agentic-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
