---
trigger: always_on
description: 一句话搞定个股分析 — 说"分析XXX"，自动采集30+数据源 → AI完成基本面(Step 0-8)+技术面+资金面的完整研报 → 生成交互式HTML报告。支持增量更新（K线/行情/技术指标自动刷新）。触发词："分析XXX股票"（首次）或"更新XXX股票"（增量）。
---


# 一句话搞定个股分析

> 版本：v3.1 | 更新：2026-05-30 | 用户说一句话 → 30+数据源采集 → Step 0-8 基本面 + 第9章技术面 + 资金面 → 双主题交互式HTML

---

## 📋 快速导航

- [⚡ 使用方式](#使用方式) — 一句话触发，全自动执行
  - [场景判断](#场景判断)
  - [场景A：首次分析](#场景a首次分析) — 完整三阶段
  - [场景B：增量更新](#场景b增量更新) — 快速刷新数据
- [🏗️ 三层架构](#三层架构) — Phase 1 数据采集 → Phase 2 AI分析 → Phase 3 HTML
- [📊 输出内容](#输出内容) — MD 报告 + HTML 报告的完整结构
- [🎨 HTML手写规范](#html手写规范) — 分批手写 + grep 机械校验
- [🔧 执行流程](#执行流程) — Phase 1/2/3 详细步骤
- [🔄 增量更新详细说明](#增量更新详细说明) — 脚本自动 + AI 手动

---

## ⚡ 使用方式

### 场景判断

**AI必须根据用户输入的关键词判断场景：**

#### 场景A：首次分析（完整流程）

**触发关键词：**
- "分析 XXX股票"
- "个股分析 XXX"
- "研究 XXX"
- "生成 XXX报告"
- 用户明确说"首次分析"

**判断逻辑：**
- 用户输入包含上述关键词 → 执行首次分析（Phase 1→2→3）

#### 场景B：增量更新（只更新数据）

**触发关键词：**
- "更新 XXX股票"
- "刷新 XXX报告"
- "增量更新 XXX"
- "更新报告"
- 用户明确说"更新"

**判断逻辑：**
1. 用户输入包含上述关键词
2. 检查 `output/个股研究-{股票名称}.html` 是否存在
3. 如果存在 → 执行增量更新
4. 如果不存在 → 提示用户："未找到 XXX 的报告，是否需要首次分析？"

---

### 场景A：首次分析

**当用户说"分析 XXX股票"时，一句话触发全流程：**

### Phase 1: 数据采集（3-5分钟）

```bash
cd G:/vibe/my-skills/stock-analysis-enhanced
python stock_full_report.py <股票代码>
```

**示例：**
```bash
python stock_full_report.py 600519  # 贵州茅台
python stock_full_report.py 600673  # 东阳光
```

**唯一输出：** `output/data_<股票代码>.json`（包含K线、财务、股东、业务构成等完整数据，供Phase 2/3使用）

**⚠️ Phase 1完成后立即进入Phase 2**，不要停下来询问用户。JSON只是数据中间产物。

**⚠️ Phase 1 新增功能：** 自动计算技术指标（MACD/KDJ/RSI/BOLL/MA），保存到 `data_{代码}.json` 的 `technical_indicators` 和 `technical_signals` 字段，供 Phase 3 生成第9章使用。

---

### 场景B：增量更新

**执行命令：**

```bash
cd G:/vibe/my-skills/stock-analysis-enhanced副本
python update_stock_report.py <股票代码>
```

**常用参数：**
```bash
# 基本用法（自动查找HTML文件）
python update_stock_report.py 000066

# 指定HTML文件
python update_stock_report.py 000066 --html output/个股研究-中国长城.html

# 强制更新（跳过检查，直接更新）
python update_stock_report.py 000066 --force
```

**更新内容：**
- ✅ K线数据（rawData数组）
- ✅ Hero区域（最新价、涨跌幅）
- ✅ 第9章技术指标（MACD/KDJ/RSI/BOLL图表）
- ❌ 基本面分析（Step 0-8，不更新）

**详细说明：** 见本文件末尾"[🔄 增量更新详细说明](#增量更新详细说明)"章节

---

### Phase 2: AI深度分析（30-60分钟）

**AI必须手动执行以下步骤：**

1. **读取数据：** `output/data_<股票代码>.json`
2. **执行MCP搜索：** 根据实际需要搜索（通常3-6次）
   - 行业趋势、市场规模
   - 竞争对手、市场份额
   - 券商研报、目标价
   - 最新新闻、政策动态
3. **逐步完成Step 0-8分析：** 严格按照分析框架，每个Step达到要求
4. **输出MD报告：** `output/个股研究-<股票名称>.md`

**⚠️ 三阶段连续执行，中间不中断：** Phase 2完成后**立即进入Phase 3**，不要停下来询问用户是否继续。MD报告只是中间产物，HTML才是最终交付物。

---

### Phase 3: HTML生成层（AI手动分批拼装 + 逐批机械校验）

> ⚠️ 写 HTML 只用 Write 工具。整个 Phase 3 只需 6 次 Write 调用，约 30 分钟，完全在单会话内完成。不要用 Python 替代。

**原理：** AI手写完整HTML——从范例提取结构参照，从 `shared/` 搬运CSS/JS，从MD报告提取内容。**分批 Write，每批写完后立即用 grep 对范例做机械校验，通过才继续。**

**核心规则：不靠记范例，靠每批 grep diff。**

**执行步骤：**

1. **读取模板：** `shared/template_base.css` + `shared/template_base.js`
2. **读取范例并提取参照：** `examples/个股研究-中国长城.html` → 用 grep 提取 nav 锚点列表、section id 列表，保存为临时参照文件
3. **读取校验规范：** `HTML手写参考.md` → 严格按「分批机械校验」表格逐批执行
4. **读取数据：** `output/个股研究-{股票名称}.md` + `output/data_{代码}.json`
5. **分批手写HTML（每批 = Write + grep校验）：**
   - 每批 ≤300行，用 **Write工具** 直接写HTML markup
   - **写完后立即跑该批对应的 grep 校验命令**（命令清单见 `HTML手写参考.md` § 分批机械校验）
   - 校验通过 → 下一批。校验失败 → grep 范例对应行 → 比对 → Edit 修正 → 重新校验
6. **合并：** 仅用 `cat` 做机械字节拼接
7. **终验：** 运行合并后终验命令（nav-id 配对 + 结构完整性），全部通过才输出最终文件
8. **输出：** `output/个股研究-{股票名称}.html`

### ⚠️ 分批手写规则（强制）

**禁止行为：**

| 禁止 | 原因 |
|------|------|
| ❌ **Python f-string/template 生成HTML** | 需要大量 `{{}}` 转义，易出错不可维护 |
| ❌ **Bash heredoc 生成HTML** | 多行HTML下经常EOF匹配失败 |
| ❌ **一次性写出整个HTML** | >300行文件必须分批，每批独立一个Write调用 |
| ❌ **用 `python -c` 拼接HTML字符串** | 属于Python生成，不是手写 |

**正确做法：**

| 操作 | 工具 | 说明 |
|------|------|------|
| 写入HTML内容 | **Write** | 直接写HTML markup，无转义问题 |
| 精确局部修改 | **Edit** | 替换特定行，不改动其他部分 |
| 机械合并部分文件 | **Bash `cat`** 或 **Python `file.read()`** | 仅读取+拼接字节，不生成任何markup |

**标准分批方案（~900行HTML）—— 每批三步：Read范例 → Write → grep：**

**参考范例：** `examples/个股研究-中国长城.html`（864行，金标准成品）

> ⚠️ **参考范例路径不可变。** 该文件位于 `examples/` 目录（稳定范例），禁止替换为 `output/` 下的生成结果。行号基于当前版本，如范例更新需同步修正行号。

| 批次 | Step 1: Read范例（先看再写） | Step 2: Write | Step 3: grep兜底 |
|------|---------------------------|---------------|------------------|
| Batch 0 | `Read 中国长城 line=1-129`（head+style段） | Write DOCTYPE + `<head>` + `<style>` | `grep -c '<style>'`=1, `grep 'echarts\|MathJax'` 确认CDN |
| Batch 1 | `Read 中国长城 line=130-195`（nav+hero+conclusion+profile网格） | Write `<body>` + nav + hero + conclusion(含交易参数行) + grid-2(公司画像+近期动态) | `grep -oc 'href="#'`=14, `grep -c 'hero-meta-item'`=5, `grep 'nav-analysis-date'`存在, `grep -c '利好\|利空\|中性'`≥5, `grep 'overflow-y\|max-height:300'`=0, `grep -c 'style="[^"]*color:#fff[^"]*"\|style="[^"]*color:white[^"]*"\|style="[^"]*color:rgba(255'`=0, `grep -c '买入区间\|止损位\|目标价\|建议仓位'`≥4 |
| Batch 2 | `Read 中国长城 line=196-260`（饼图+财务+K线） | Write 饼图+财务grid-2 + K线card(kline-section) | `grep 'id="kline-section"'` 存在, `grep 'height:480px'` 存在, `grep -c 'markLine'`≥1, `grep -c 'markPoint'`≥1 |
| Batch 3 | `Read 中国长城 line=260-397`（Step 0-3） | Write Step 0-3 卡片（mission/macro/chain/quality） | `grep -o 'id="[^"]*"'` → mission/macro/chain/quality 四者存在 |
| Batch 4 | `Read 中国长城 line=398-550`（Step 4-6） | Write Step 4-6 卡片（elasticity/risk/valuation） | `grep -o 'id="[^"]*"'` → elasticity/risk/valuation 三者存在, `grep -c 'score-fill.*width'`≥10, `grep 'border-top:2px solid var(--gold)'` 弹性树横线存在 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mingli30119/stock-analysis](https://github.com/mingli30119/stock-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
