---
trigger: always_on
description: |
---


# academic-search Skill

## 前置检查

在开始前，检查环境就绪状态：

```bash
bash ~/.claude/skills/academic-search/scripts/check-deps.sh
```

- **Node.js 22+**：必需（用于 CDP 浏览器模式）。仅使用 API 平台时可不检查。
- **Chrome remote-debugging**：仅在访问 Google Scholar 或其他需要浏览器自动化的平台时必需。在 Chrome 地址栏打开 `chrome://inspect/#remote-debugging`，勾选 **Allow remote debugging for this browser instance**。
- **curl**：必需，用于 API 调用。

arXiv、Semantic Scholar、PubMed、Papers with Code 等 API 平台无需 Chrome 远程调试即可使用。

**S2 API Key（强烈建议）**：无 Key 时 S2 速率上限极低，单 session 多次调用必触发 429。免费注册即可获得更高配额：https://www.semanticscholar.org/product/api#api-key-form。有 Key 时在请求头加 `x-api-key: {your_key}`。

## 搜索哲学

**明确目标，选对平台，提取结构化数据，完成即止。**

学术搜索不同于通用网页浏览——目标是获取**准确、结构化**的论文元数据，而不是浏览网页内容。

**① 明确检索目标，定义成功标准**：执行前先明确什么算完成了。

- 关键词搜索？精确论文？某作者的全部论文？某 venue 的论文列表？
- 学科是什么？是否需要使用 MeSH、JEL、MSC、ACM CCS 等受控词表？
- 文献类型是什么：期刊论文、会议论文、预印本、系统综述、临床试验、工作论文、专著/章节？
- 需要什么字段：仅标题和引用数 / 完整元数据 / PDF / BibTeX / 代码链接？
- 年份范围？领域限定？返回几篇？
- **成功标准**：用户要的是摘要表（第一遍）还是完整元数据（第二遍）？数量够了吗？字段都有了吗？这是后续所有决策的锚点。

**② 选对平台**：不同需求对应不同平台（见下方矩阵）。API 平台优先，CDP 用于无 API 的平台。

**③ 提取结构化数据，先筛后深**：搜索的时间瓶颈不在"搜"，在"筛"。默认采用两遍策略：

- **第一遍（轻量扫描）**：先拉 20-30 条结果，输出轻量摘要表——标题、作者、年份、venue、引用数、是否有开放 PDF/代码。不拉完整摘要。
- **用户或任务确认核心论文**（引用数高、venue 等级高、与目标最相关的 5-10 篇）后，**第二遍**再深入拉摘要、PDF、BibTeX 等完整信息。

所有结果输出为统一 schema（见 `references/metadata-schema.md`），不要输出原始 HTML 或非结构化文本。多平台结果用 DOI/arXiv ID 去重合并。

**④ 过程校验，用失败信号更新方向**：每一步的结果都是信息，不只是成功或失败的二元信号。

| 失败信号 | 含义 | 方向调整 |
|---------|------|---------|
| API 429 / Rate exceeded | 本次会话消耗超配额，不是暂时波动 | 等待 15s+ 或切换 CDP 模式；不要同一请求重试 |
| Jina/WebFetch 超时 | 该页面对静态抓取不友好 | 改用 curl 直接调 API 或切换 CDP |
| S2 返回结果为空 | query 措辞问题，或该平台无收录 | 换关键词组合，或换 arXiv/PubMed |
| 平台返回"内容不存在" | 未必真的不存在，可能是访问方式问题 | 检查 URL 参数是否完整，换平台验证 |
| 同一方式重试 3 次无改善 | 路径错了，不是还没找到方法 | 重新评估目标，换平台或换访问方式 |

**⑤ 完成判断**：对照①定义的成功标准确认任务完成后停止，不为"更完整"而过度操作。

## 平台选择矩阵

根据任务特征选择最合适的平台和访问方式：

| 需求 | 首选平台 | 访问方式 | 备注 |
|------|---------|---------|------|
| CS/Math/Physics/统计 论文搜索 | **arXiv** | REST API | 完全开放，PDF 直链 |
| 引用数、引用/被引关系 | **Semantic Scholar** | REST API | 免费 Key 可提升速率 |
| 作者主页、全部论文 | **Semantic Scholar** | REST API | /author/{id}/papers |
| 生物医学、生命科学 | **PubMed** | NCBI E-utilities | 完全开放 |
| 跨学科 DOI / 元数据核对 | **Crossref** | REST API | DOI、期刊、出版商、ISSN、参考文献基础信息 |
| 跨学科作者/机构/概念/引用 | **OpenAlex** | REST API | 适合作为 Semantic Scholar 的跨学科补充 |
| 开放获取状态 / OA PDF | **Unpaywall** | REST API | 判断 gold/green/hybrid/closed OA 与合法开放全文 |
| ML 论文 + 代码仓库 | **Papers with Code** | REST API | 无需鉴权 |
| ACM 顶会论文 (SIGKDD/WWW 等) | **ACM DL** | WebFetch + Jina | BibTeX 导出端点可直接访问 |
| IEEE 期刊/会议论文 | **IEEE Xplore** | WebFetch / Jina | 有机构 Key 时用官方 API |
| 广泛引用数 / 全平台覆盖 | **Google Scholar** | **CDP（必须）** | 无 API，反爬严重 |
| 论文是否存在 / 基础元数据 | **Semantic Scholar** | REST API | 支持 DOI / arXiv ID 互查 |
| **中文文献**（期刊/学位论文/会议） | **CNKI（知网）** | **CDP（必须）** | 无公开 API；机构登录后全文可得 |

**API 平台访问方式**：

- **WebSearch**：用于发现论文来源、查找 DOI/作者 ID 等信息入口
- **WebFetch / Jina**：URL 已知时从页面提取，Jina（`r.jina.ai/{url}`）节省 token，适合文章类页面
- **curl**：直接调用结构化 API，返回 JSON/XML
- **CDP**：仅 Google Scholar 必须；其他平台在 API/WebFetch 无效时作为兜底

详细 API 调用模板见 `references/api-cookbook.md`。

## 学科路由

先按用户问题判断学科，再读取对应 `references/disciplines/*.md`。如果用户问题跨学科，优先读取最核心学科的 profile，再用 OpenAlex / Crossref / Unpaywall 做跨学科补全。

| 学科 | 读取文件 | 首选方向 |
|------|----------|----------|
| 计算机 / AI | `references/disciplines/computer-science.md` | arXiv、Semantic Scholar、ACM DL、IEEE、DBLP、Papers with Code |
| 医学 / 生命科学 | `references/disciplines/biomedicine.md` | PubMed、PMC、Europe PMC、ClinicalTrials、bioRxiv、medRxiv |
| 物理 / 数学 | `references/disciplines/physics-math.md` | arXiv categories、NASA ADS、INSPIRE HEP、MSC |
| 化学 / 材料 | `references/disciplines/chemistry-materials.md` | Crossref、OpenAlex、ChemRxiv、ACS、RSC、Springer、Wiley |
| 经济 / 社科 | `references/disciplines/economics-social-science.md` | RePEc、NBER、SSRN、OSF、PsyArXiv、JEL |
| 人文 / 法律 | `references/disciplines/humanities-law.md` | Google Scholar、图书馆目录、JSTOR/Project MUSE/HeinOnline 访问状态 |

学科 profile 决定 query expansion、排序标准、输出字段和全文访问边界。不要把 CCF 或 CS 顶会规则套到非 CS 学科。

## 核心能力

### 关键词搜索

1. 先按“学科路由”读取 discipline profile：CS/ML → arXiv + Semantic Scholar；生医 → PubMed/Europe PMC；跨领域 → OpenAlex + Crossref + Semantic Scholar
2. **扩展 query**：用户自然语言输入往往只是一个切入点，需要主动展开为 2-3 个互补 query 覆盖不同命名习惯：
   - 同义词替换：`agent` → `agentic` / `multi-agent` / `autonomous`
   - 子概念拆分：`time series agent` → `time series LLM agent` + `time series agentic reasoning` + `time series automated analysis`
   - 缩写与全称并用：`TS` / `time series`，`LLM` / `large language model`
   - 学科受控词表：医学用 MeSH，经济用 JEL，数学用 MSC，计算机用 ACM CCS，化学可补 CAS/化合物同义词
   - 不同 query 结果合并去重，覆盖率比单 query 提升 30-50%
3. 构造查询：arXiv 用 `search_query` 字段前缀语法；S2 用 `query` 参数；PubMed 用 `term` 布尔表达式
4. **计划多次 S2 调用时优先用 batch API**（`/paper/batch`）而非多次 search，节省速率配额
5. **第一遍输出轻量摘要表**（必含：标题、年份、venue、引用数、是否有开放 PDF），**不默认拉完整摘要**
6. **意图判断**：用户明确说"只要前 N 篇"或"摘要表即可"时，直接输出第一遍结果，无需等待确认再停下
7. 用户需要第二遍时，再深拉完整元数据

多平台并行查询时，用子 Agent 分治（见"并行分治策略"一节）。

**轻量摘要表输出格式示例**：

| 标题 | 年份 | Venue | 引用数 | PDF |
|------|------|-------|--------|-----|
| Attention Is All You Need | 2017 | NeurIPS [CCF-A] | 120,000+ | ✓ arXiv |
| BERT: Pre-training... | 2019 | NAACL [CCF-B] | 80,000+ | ✓ arXiv |

Venue 等级标注规则：CS 会议参考 `references/venue-rankings.md`（CCF 分级）；非 CS 学科先读取 `references/disciplines/*.md` 和 `references/rankings/*.md`，按该学科的证据等级、文献类型或期刊/来源规则排序。期刊显示 JCR 分区（若可从平台字段获取）时必须标明来源。

### 结果筛选


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ustc-ai4science/academic-search](https://github.com/ustc-ai4science/academic-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
