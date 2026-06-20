---
trigger: always_on
description: >
---


# AI Daily Research

自动采集、分析并生成每日 AI 新闻日报 + 论文深度研究的 Hermes Agent Skill。

## 架构（v2 — 存储原文）

```
Phase 1: 采集+存原文 (Python脚本)
├── 新闻: RSS/API → JSON (标题+摘要, 不存原文)
└── 论文: PDF提取/摘要 → 存 raw/papers/{date}/{id}.txt
                                          ↓
Phase 2: 选+读原文+分析 (LLM)
├── 读轻量元数据JSON → 按重要性选6条新闻 + 2篇论文
├── 读选中论文的 raw/papers/{date}/{id}.txt 原文
├── 基于原文生成深度分析
└── 写飞书 + Wiki
```

**关键设计**：脚本存原文到磁盘，LLM按需读取（不塞全部进context window）。新闻只看摘要，论文读全文。

## ⚠️ 核心原则（最高优先级，覆盖所有场景）

### 原则一：论文获取优先
无论用户以何种方式提供论文（PDF附件、URL链接、口头提及），
**第一步永远是确保能获取并保存论文原文到磁盘**（存入 `~/wiki/raw/papers/`）。
没有原文，后续分析无法进行，也无法复用。

论文原文获取优先级：
1. PDF附件 → PyMuPDF 提取全文（脚本，不消耗 token）
2. arXiv URL → 下载 PDF → PyMuPDF 提取全文
3. OpenReview/其他学术URL → 抓取页面 → 提取摘要 + 全文
4. 公众号/网页URL → **Python 直接提取文字**（curl + BeautifulSoup，不下载 PDF）
5. 口头提及 → 搜索论文 → 按上述方式获取

**如果获取失败：** 告知用户获取失败原因，建议用户提供 PDF。不跳过获取步骤。

### 原则二：获取即保存，分析需确认
- **获取原文后立即保存**到磁盘（`~/wiki/raw/papers/`），确保不丢失
- **5维度深度分析需要用户确认**后才执行（省 token）
- 分析完成后**必须**同时写入 `analyzed_sources.json` + Wiki

**流程：** 获取原文 → 保存 → 问用户 → 用户选"深度分析" → 分析 + 存储
**不要：** 获取原文 → 自动开始分析（浪费 token）

### 原则三：查询优先检索 Wiki
当用户提问涉及某个主题/论文/概念时，
agent 在回答前**必须**先检查：
1. `analyzed_sources.json` — 看是否已分析过相关论文
2. `~/wiki/concepts/` — 看是否有相关概念页
3. `~/wiki/entities/` — 看是否有相关实体页

如果找到已有分析 → **直接使用已有分析，不重新分析全文**。
如果未找到 → 按正常流程分析 + 存储。

**检索命令：**
```bash
# 按关键词搜索已分析论文（标题/标签/摘要）
python3 -c "
import json, sys
with open('/opt/data/cron/output/analyzed_sources.json') as f:
    data = json.load(f)
kw = sys.argv[1].lower()
for p in data.get('papers', []):
    if kw in p.get('title','').lower() or kw in ' '.join(p.get('tags',[])).lower() or kw in p.get('brief','').lower():
        print(f\"  {p['arxiv_id']}: {p['title']} [{', '.join(p.get('tags',[]))}]\")
" "搜索关键词"

# 搜索概念页（文件名 + 内容）
grep -ril "关键词" ~/wiki/concepts/ 2>/dev/null

# 搜索实体页
grep -ril "关键词" ~/wiki/entities/ 2>/dev/null
```

### 原则四：LLM 只读需要的部分（省 token 关键）
**PDF/URL → 文本提取是脚本操作（不消耗 token）。只有 LLM 读取文本时才消耗 token。**

因此必须按操作类型控制 LLM 读取量：

| 操作 | LLM 需要读什么 | 最大字符数 | 说明 |
|------|---------------|-----------|------|
| **Step 1.8 元数据提取** | 前 5000 字符 | 5000 | 只需标题、作者、摘要 |
| **Option 2 只看摘要** | 摘要 + 结论 | 8000 | 前3000 + 后3000 + 扫描中间 |
| **Option 1 深度分析** | 全文（智能截断） | 30000 | 跳过附录/参考文献，读核心章节 |
| **日报 cron** | 只读选中的 2 篇 | 各 30000 | 不读全部候选论文 |

**智能截断策略（深度分析时）：**
```python
def smart_truncate(full_text, max_chars=30000):
    """智能截断：保留核心章节，跳过附录/参考文献"""
    # 找到各章节位置
    sections = {}
    for marker in ['abstract', 'introduction', 'method', 'methodology',
                   'experiment', 'results', 'conclusion', 'discussion',
                   'references', 'appendix']:
        idx = full_text.lower().find(marker)
        if idx >= 0:
            sections[marker] = idx

    # 保留: abstract → conclusion（跳过 references/appendix）
    end_markers = ['references', 'bibliography', 'appendix', 'supplementary']
    end_idx = len(full_text)
    for m in end_markers:
        if m in sections:
            end_idx = min(end_idx, sections[m])

    truncated = full_text[:end_idx]

    # 如果还超长，优先保留 abstract + method + results + conclusion
    if len(truncated) > max_chars:
        # 保留前5000（abstract+intro）+ 中间方法结果 + 后5000（conclusion）
        head = truncated[:5000]
        tail = truncated[-5000:] if len(truncated) > 5000 else ""
        middle_start = 5000
        middle_end = min(len(truncated), max_chars - 10000)
        middle = truncated[middle_start:middle_end]
        return head + "\n[...截断...]\n" + middle + "\n[...截断...]\n" + tail

    return truncated
```

**⚠️ 关键：不要把整篇论文原文直接塞进 context！** 按上表控制读取量。
- 论文原文: `/opt/data/cron/raw/papers/{YYYY-MM-DD}/{arxiv_id}.txt`
- 元数据JSON: `/opt/data/cron/output/ai_raw_{YYYYMMDD}.json`

### 元数据JSON中论文字段
| 字段 | 说明 |
|------|------|
| `title` | 论文标题 |
| `abstract` | 摘要（内联） |
| `raw_text_path` | 原文文件路径（LLM用这个读全文） |
| `authors` | 作者+单位 |
| `source` | 来源(arXiv/OpenAlex/DBLP等) |

⚠️ **不再有 `full_text` 字段！** 论文全文存在 `raw_text_path` 指向的文件中。

## 数据源

| 类型 | 来源 | 备注 |
|------|------|------|
| 🇨🇳 中文 | 雷峰网/量子位/极客公园/钛媒体/IT之家/36氪 | 36氪限速 |
| 🌍 英文 | Google News(重试机制)/HN(points>30) | |
| ✍️ 博客 | 11个Karpathy推荐顶级技术博客(Simon Willison/GWERN/Paul Graham等) | RSS |
| 📄 学术 | arXiv(PDF全文)+OpenAlex(作者单位)+DBLP(会议) | |
| 📚 出版 | CrossRef(DOI解析,覆盖非arXiv期刊/会议) | 覆盖最广 |
| 📝 顶会 | OpenReview(NeurIPS/ICLR/ICML poster+oral) | 有完整摘要 |

## 大厂论文优先

脚本标记 is_focus_company，论文排序时大厂优先。

**国际**: OpenAI, Google DeepMind, Anthropic, Meta AI, Microsoft, Apple, Amazon, NVIDIA, xAI, Mistral
**国内**: 百度, 阿里, 腾讯, 字节, 华为, 美团, 小米, 商汤, Kimi, 智谱AI, DeepSeek, MiniMax, 蚂蚁

## 增强技术（借鉴 hermes-arxiv-agent）

### PDF 机构提取增强
从 PDF 前2页提取作者单位，包含：
- **ORG_KEYWORDS**: 100+ 机构关键词（Google DeepMind, MIT, 清华, 北大等）
- **CamelCase 分词**: `DepartmentofCS` → `Department of CS`
- **跨行连字符合并**: `Repub-` + `licof Korea` → `Republic of Korea`
- **噪音过滤**: URL、邮件、公式、正文片段
- **启发式判断**: `looks_like_affiliation()` 智能判断是否为真实机构信息

### Excel 持久化记录
- `papers_record.xlsx` 存储所有已处理论文
- 支持 upsert（按 arxiv_id 更新，不重复插入）
- 质量排序：优先保留摘要+单位+日期更完整的记录

### Pending 队列（安全重试）
- `pending_llm_ids.txt` 跟踪 LLM 未完成的论文
- 脚本输出 `[LLM_SUMMARIZATION_REQUIRED]` 标记
- 支持中断后安全恢复

## 输出格式

- 📰 AI 要闻 6条（精选，不分国内外）
- 📄 论文 2篇（大厂优先，含全部作者单位 + 创新点深度解析5维度）
- 📊 趋势 1句话

## 论文去重机制

为避免重复分析同一篇论文，使用 JSON 文件记录已处理的论文：

### 记录文件位置
`/opt/data/cron/output/analyzed_papers.json`

### 文件格式
```json
{
  "last_updated": "2026-04-30",
  "papers": [
    {
      "arxiv_id": "2604.26649",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CODE-BULIAO/ai-research-wiki](https://github.com/CODE-BULIAO/ai-research-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
