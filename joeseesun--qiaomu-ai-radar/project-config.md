---
trigger: always_on
description: **问题**：v2.14 后台 agent 仍用逐条 Edit（50+ 次 API 调用，~5 分钟）
---

# Daily Topic Selector - 架构文档

> 每日选题助手的系统架构与设计哲学

## 🔖 版本历史

### v2.15 (2026-03-04) - Read→Write 极速过滤翻译 ⚡

**问题**：v2.14 后台 agent 仍用逐条 Edit（50+ 次 API 调用，~5 分钟）

**方案**：Read 一次 → 内存中过滤+翻译 → Write 一次（2 次调用，~10 秒）

**设计哲学**：批量 > 逐条，内存处理最快，消除工具调用开销

### v2.10 (2026-01-10) - AI相关性智能过滤 🧹

**问题**：HN等综合平台全吞模式，铁路/游戏/政治等无关内容进入周刊

**方案**：延续v2.7哲学——脚本采集，Claude过滤

**工作流**：`采集 → 智能过滤（新增）→ 翻译 → 阅读`

**过滤标准**：
- 保留：AI/ML、编程、创业、认知科学、设计、科技动态
- 删除：政治、体育、纯硬件、交通基建、娱乐八卦

**零代码改动**：过滤逻辑在SKILL.md工作流中定义，由Claude执行

### v2.8 (2026-01-10) - 新增 8 个 AI/商业信息源 📰

**新增信息源**（每源限制3篇最新文章）：

**Substack 类**（使用通用 `substack_generic.py` 解析器）：
1. **DC The Median** - 数据科学与AI应用洞察
2. **Mark McNeilly** - AI新闻周报 "The New News in AI"
3. **Business Analytics** - 商业分析与数据驱动决策
4. **AI Leadership Edge** - AI领导力与企业战略
5. **ChinAI Newsletter** - 中国AI发展追踪（Jeff Ding）
6. **Why Try AI** - AI工具测评与实操指南

**Beehiiv 类**（使用通用 `beehiiv_generic.py` 解析器）：
7. **The Rundown AI** - 每日AI新闻速递（百万订阅）
8. **The Neuron Daily** - AI新闻与行业洞察

**架构变更**：
```
新增：scripts/parsers/substack_generic.py (80行)
  - 通用 Substack RSS 解析器
  - 自动将基础URL转换为/feed端点
  - 工厂函数：parse_dcthemedian, parse_markmcneilly 等

新增：scripts/parsers/beehiiv_generic.py (70行)
  - 通用 Beehiiv RSS 解析器
  - 处理 rss.beehiiv.com/feeds/xxx.xml 格式
  - 工厂函数：parse_therundown, parse_theneuron

修改：config/sources.json
  - 新增8个信息源配置
  - 每源 limit: 3（只抓最新3篇）
  - check_interval_hours: 24（每日检查）

修改：scripts/fetch_updates.py
  - 导入新解析器模块
  - PARSERS字典注册8个新解析器
```

**设计哲学**：
- **消除重复**：Substack/Beehiiv 各用一个通用解析器，新源只需配置
- **工厂模式**：`parse_xxx()` 函数只是调用 `parse_substack_rss(source)`
- **单一真相源**：URL和参数在 `sources.json`，代码只负责解析

### v2.7 (2026-01-04) - 周刊工作流 + 零API翻译 📅

**核心变更**：从日志模式切换到周刊模式，翻译由Claude直接完成

**周刊工作流**：
1. **采集模式改变**：
   - 从 `content_log.md`（日志追加）→ `{year}-第{week}周.md`（按周组织）
   - 采集时自动写入本周文件，按ISO周数组织
   - 新文章按日期分组（`## 2026-01-04`）

2. **翻译工作流重构**：
   - **删除**：`llm_utils.py`（114行API调用代码）
   - **原理**：能用LLM直接解决的事，不要写脚本
   - **新流程**：
     1. 采集脚本只记录原始数据（英文标题、中文标题都保留）
     2. Claude读取文件后用Edit工具批量添加翻译
     3. 格式：`- **标题**: English Title / 中文翻译`
   - **收益**：零API调用、零依赖、零token成本

3. **"值得写"索引**：
   - 新增 `generate_worth_writing.py`
   - 从 `state.json` 提取标记为 `📒 to_write` 的文章
   - 生成 `值得写.md` 供Claude分析和推荐
   - 自动在采集流程后更新

**文件变化**：
```
新增：scripts/generate_worth_writing.py (86行) - 生成"值得写"索引
新增：scripts/update_indexes.py (67行) - 一键更新所有索引
修改：scripts/fetch_updates.py
  - append_to_log() → append_to_weekly_log()
  - 简化翻译逻辑（114行API代码→0行）
  - 移除自动索引生成（职责分离）
删除：scripts/shared/llm_utils.py (不再需要)
```

**设计哲学的胜利**：
- **简单消灭复杂**：114行API代码 → 0行，翻译质量更好
- **单一职责**：脚本负责数据，Claude负责智能
- **消除边界情况**：
  - 无需处理API超时、重试逻辑
  - 无需处理中英文混合的edge case
  - 无需维护翻译prompt模板
- **好品味的代码**：让人说"操，原来可以这么简单"

**工作流示例**：
```bash
# === 自动部分（采集） ===
python3 fetch_updates.py
# 1. 同步上次的emoji标记 → state.json
# 2. 采集新文章 → 2026-第1周.md
# 3. 提示下一步操作

# === 手动部分（处理） ===
# 1. Claude批量翻译周刊标题
#    Read → Edit → "English Title / 中文翻译"

# 2. 用户阅读周刊，打emoji标记
#    ⭐️ 收藏  📒 值得写  ✅ 已读  ❌ 跳过

# 3. 一键更新所有索引
python3 update_indexes.py
# → 同步标记 → 更新收藏索引 → 更新值得写索引
```

**文件职责分离**：
- `fetch_updates.py` - **只采集**，不生成索引
- `update_indexes.py` - **只更新索引**（打标后运行）
- 收藏索引/值得写索引 - **跨周累积**，不随周刊清空

**关键理解**：
- 周刊（`2026-第1周.md`）= **Inbox**（本周待处理，下周清空）
- 收藏索引（`⭐️收藏精选.md`）= **Archive**（永久保存，跨周累积）
- 值得写（`值得写.md`）= **创作清单**（待创作的选题）

### v2.6 (2026-01-04) - 新增 4 个顶级播客源 🎙️

**新增播客源**：
1. **Lex Fridman Podcast** (lexfridman.com)
   - AI、科学、哲学深度访谈
   - 嘉宾包括顶级研究者、创业者、思想家
   - 长篇对话（2-4小时），深度探讨

2. **Cognitive Revolution** (cognitiverevolution.ai)
   - AI 前沿技术与应用深度解析
   - AI 创业案例与产业洞察
   - 技术趋势前瞻

3. **80,000 Hours Podcast** (80000hours.org)
   - 有效利他主义（Effective Altruism）
   - 职业选择与社会影响力
   - AI 安全、全球优先事项

4. **Latent Space** (latent.space)
   - AI 工程实践与产品开发
   - AI 创业公司深度访谈
   - 技术栈选择与架构设计

**技术实现**：
- 创建 4 个 podcast parser：`lexfridman.py`, `cognitiverevolution.py`, `hours80k.py`, `latentspace.py`
- 全部使用 RSS feed（零依赖，简单可靠）
- Latent Space 初始使用 Flightcast URL 超时，改用 Substack `/feed` endpoint 成功
- 测试通过，成功采集 40 篇节目（10+10+10+10）

**内容矩阵再扩展**：
| 维度 | v2.5 覆盖 | v2.6 新增播客 |
|------|----------|-------------|
| AI 技术深度 | 袁超发、HF、Import AI | **Lex Fridman**（顶级对话）<br>**Cognitive Revolution**（前沿技术） |
| AI 工程实践 | - | **Latent Space**（工程与创业） |
| AI 安全/伦理 | - | **80,000 Hours**（有效利他） |
| 跨学科视野 | Wait But Why | **Lex Fridman**（科学/哲学） |

**采集内容示例**：
- Latent Space: "The Agent Labs Thesis", "Agentic Commerce Protocol"
- Lex Fridman: 对话顶级 AI 研究者与创业者
- Cognitive Revolution: AI 应用案例深度分析
- 80,000 Hours: AI 安全与职业影响力

**设计哲学**：
- **播客优先 RSS**：所有 podcast 都有标准 RSS，稳定可靠
- **URL 调试**：Latent Space 从 Flightcast → Substack，找到最稳定endpoint
- **零额外依赖**：继续复用 `web_utils.py` 的 `fetch_html()`

### v2.5 (2026-01-04) - 新增 4 个高质量信息源 🌐

**新增信息源**：
1. **袁超发技术博客** (yuanchaofa.com)
   - LLM/AI 技术深度文章
   - 手写大模型组件系列（Transformer、LoRA、GQA等）
   - 中文技术内容，工程实践导向

2. **Farnam Street Brain Food** (fs.blog)
   - 认知科学、心理学、决策理论
   - 225+ 期周更 newsletter
   - 深度思考与智慧洞察

3. **Austin Kleon** (austinkleon.substack.com)
   - 创意写作与艺术实践
   - "Steal Like an Artist" 作者
   - 创作过程与生活洞察

4. **Paul Graham Essays** (paulgraham.com)
   - 创业哲学与黑客文化
   - Y Combinator 创始人
   - 经典长文（How to Do Great Work 等）

**技术实现**：
- 创建 4 个新 parser：`yuanchaofa.py`, `brainfood.py`, `austinkleon.py`, `paulgraham.py`
- 3 个使用 RSS feed（简单可靠），1 个 HTML 解析（Paul Graham 无 RSS）
- 更新 `sources.json` 和 `fetch_updates.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joeseesun/qiaomu-ai-radar](https://github.com/joeseesun/qiaomu-ai-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
