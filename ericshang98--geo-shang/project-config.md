---
trigger: always_on
description: Use when optimizing a website for AI search engines (GEO) and traditional search engines (SEO). Starts by checking for a positioning document (geo-shang.html), then audits, researches keywords, optimizes, and records changes. Triggers on "geo", "seo", "geo-shang", "AI search optimization", "why can't AI find my site", or any request to improve search visibility. Consumes significant resources — deep research, parallel subagents, exhaustive analysis.
---


# GEO Shang — Generative Engine Optimization

与用户对话使用中文。geo-shang.html 定位文档使用中文（给用户审阅的）。面向外部的内容（llms.txt、meta 标签、页面正文、JSON-LD 等搜索引擎和用户看到的）使用英文。

## 文档结构：两区制（CRITICAL）

`geo-shang.html` 里有两种生命周期完全不同的内容，**必须用一条醒目的护栏分隔，绝不混排**：

- **✅ 创始人确认区（顶部）** — 定位内容：核心身份、核心信息、价值主张、功能排序、创始人身份标签、目标搜索词、Tone。**一经创始人确认，即为对外营销的唯一真源，长期稳定，可直接照做。** 写这区时假设"读者要拿去做内容营销"。
- **⚠️ 审计区（底部）** — 当前状态诊断：虚假声明必删表、7 维优先级工单表、可见度快照、归因表。**这是一份活的工单看板，不是一次性报告**——它实时反映"此刻网站与确认定位之间的差距"。每次运行、每轮优化都要就地更新：修掉的删、状态变的改、新发现/新页面/新文案重新过一遍。它不是营销内容，是给"来修 bug 的 agent"看的；对应问题修完即从本区销账。详见 A.3。

两区之间放一条视觉护栏（如 `<hr>` + 醒目标题 + 一句说明）。审计区每个标题必须自带"临时 / 非营销 / 修完即删"的字样，让任何人扫一眼就知道这不是确认的定位内容。

判断归属：**"创始人确认即真理、可直接做营销"的进确认区；"当前现状有问题、待修、改完作废"的进审计区。** 拿不准时问自己：这条修好之后还留着吗？留着 → 确认区；删掉 → 审计区。

## First Step: Check for Positioning Document

Everything depends on `geo-shang.html`. Start by checking if it exists:

```
Glob: **/geo-shang.html
Glob: **/geo-shang.md
```

- **Not found** → go to "When No Document Exists" below (full research + draft)
- **Found** → read it (including Change History), understand what changed and what's been done before, then decide your own scope and depth based on the content and history

---

## When No Document Exists

Tell the user: "没有找到定位文档，我先调研你的项目，然后起草一份给你审阅。"

### A.1 Project Research

Do all research yourself — don't ask the user questions one by one.

**Read everything in the project:**
- README.md, CLAUDE.md, package.json
- Website pages — homepage, about, pricing, features, blog, news
- llms.txt if it exists
- GitHub description / repo metadata

**Search externally (10-15+ searches):**
- Brand name — what does the internet say?
- Competitor products — how do they position themselves?
- Category queries — what language do customers use?
- AI engine results — what does Perplexity/ChatGPT say about this product?

**Analyze competitors (fetch 3+ competitor pages):**
- What content structure do they use?
- What claims do they make?
- Do they have llms.txt? What schema?
- What are they doing that this project isn't?

### A.2 Draft geo-shang.html

Write `geo-shang.html` in the project root. HTML format so the user can open in browser and edit directly. Each section uses `contenteditable="true"`。

文档按"两区制"组织（见上文）。本节起草的是 **✅ 创始人确认区** 的所有定位 section；审计区在 A.3 才生成，放在护栏下方。

**创始人确认区 — Required sections：**

| Section | Content |
|---|---|
| Core Identity | 一段话：这是什么、给谁用、做什么 |
| Key Message | 最重要的一句话。AI 只引用一句的话，就是这句 |
| Differentiation | 跟竞品有什么不同。具体的、事实的 |
| Value Proposition | 解决什么问题。用之前 vs 用之后 |
| Features to Highlight | 按用户重要性排序，不按技术复杂度 |
| Features to De-emphasize | 存在但不应该成为重点的功能 |
| Pricing | 具体数字 |
| Founder / Team | 背景、资历、为什么做这个。**只写创始人确认属实的，绝不编造资历/头衔/经历** |
| Target Queries | 5-10 个用户真实会搜的词 |
| Tone & Voice | 描述的风格 |
| Change History | 空的，后续每次运行追加 |

确认区结尾放一条护栏（`<hr>` + 标题如「⚠️ 以下为审计区 — 当前状态问题，临时，修完即删，非营销内容」），A.3 的审计产物全部落在护栏下方。

**Present to user:**
1. 在终端里把每个 section 内容复述给用户看
2. 给出文件的完整绝对路径，让用户可以直接点击或复制打开（例如 `file:///Users/xxx/project/geo-shang.html`），也可以用 `open` 命令帮用户在浏览器中打开
3. 等用户确认后继续

### A.3 Full Audit + Diagnosis（穷尽式）

文档确认后，做一次**穷尽式审计**——不是抽查几项，而是逐维度逐页跑完下面 7 个面，每一条都给出"现状 → 差距 → 行动"。**所有审计产物（虚假声明必删表、各维度发现、优先级表、归因表）一律写进 geo-shang.html 护栏下方的「⚠️ 审计区」，绝不混进确认区。**

> **审计区是活的，不是一次性快照。** 它反映的是"此刻网站与确认定位之间的真实差距"。每次跑 `/geo-shang`、每做完一轮优化，都要**重新核对并就地更新这张审计**：修掉的删、状态变的改、新发现的加、新建的页面/新写的文案重新过一遍可引用性。把它当成一份持续维护的 GEO 工单看板，而不是写完就不动的报告。审计区永远要能回答："如果现在有人问'网站还差什么才能被 AI 引用'，看这里就够了。"

并行铺开，能用 subagent 就并行跑：

**① 抓取与可访问性（Crawlability）— AI 爬虫能不能拿到内容**
- robots.txt：逐个 AI 爬虫 UA 核对放行情况——`GPTBot`、`OAI-SearchBot`、`ClaudeBot`、`anthropic-ai`、`PerplexityBot`、`Google-Extended`、`CCBot`、`Bytespider`、`Amazonbot`、`Applebot-Extended`。哪个被 Disallow 写明
- Cloudflare / CDN / WAF：是否开了 "Block AI Bots" 开关、是否有 Bot Fight Mode 误伤、是否对爬虫返 403/验证码
- 渲染方式：SSR/SSG vs CSR。**多数 AI 爬虫（GPTBot、ClaudeBot、PerplexityBot）不执行 JavaScript** → CSR-only 的内容对它们等于不存在。逐页判断关键内容是否在首字节 HTML 里
- HTTP 层：状态码、重定向链（有没有多跳/链路坏）、canonical 是否自指且正确、http→https、www 归一
- 速度 / Core Web Vitals（LCP/CLS/INP）— 影响 Google 排名与抓取预算

**② AI 入口文件 — 专门喂给 LLM 的**
- `llms.txt` / `llms-full.txt`：是否存在、是否对齐确认定位、是否覆盖核心页面与一句话定位、链接是否有效
- `sitemap.xml`：是否覆盖所有页面、`lastmod` 是否新鲜（3 个月断崖）、是否在 robots 里声明、是否提交搜索引擎
- favicon / OG 图 / 品牌图等机器可读资产是否齐全

**③ 结构化数据（Schema / JSON-LD）— 被引用概率 ×2.5**
- 逐页核对该有的 schema：`Organization`、`Product`/`Offer`、`Person`（创始人）、`Article`/`BlogPosting`、`FAQPage`、`BreadcrumbList`、`AggregateRating`
- 每个字段是否齐全、是否**对齐确认定位的事实**（价格、名字、状态不能和确认区打架）
- 是否能通过 Schema 校验（无语法/类型错误）

**④ Meta / 页面头（逐页）**
- 每页：`title`（长度 ≤60、含关键词+品牌、唯一不重复）、`description`（120-160、含目标词、有召唤）、`canonical`、`lang`、`viewport`
- OG（`og:title/description/image/url/type`）+ Twitter Card 是否齐全、预览是否好看
- 多语言站：`hreflang` 是否成对正确

**⑤ 内容质量与可引用性（GEO 核心，对照 Research Foundation 的硬指标）**
- **前 100 字直答**：90% 高引用页在前 100 字就给出直接答案——逐页检查 hero/首段是否直接回答"这是什么、给谁、解决什么"
- **前 30% 放关键事实**：44.2% 的 AI 引用来自页面前 30%——最关键的事实/数字有没有沉到底部

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericshang98/geo-shang](https://github.com/ericshang98/geo-shang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
