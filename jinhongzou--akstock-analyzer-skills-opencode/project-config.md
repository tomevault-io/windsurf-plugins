---
trigger: always_on
description: A股股票分析 OpenCode Skills 集合，基于 akshare 数据源。包含 18 个独立 Skill，可单独使用也可组合输出完整报告。
---

# AGENTS.md - Stock Analyzer Skills

## Repository Overview

A股股票分析 OpenCode Skills 集合，基于 akshare 数据源。包含 18 个独立 Skill，可单独使用也可组合输出完整报告。

## Key Entry Points

### Skill 调用方式
```
skill(name="stock-analyzer")         # 个股估值
skill(name="technical-analyzer")     # 技术分析
skill(name="a-dividend-analyzer")   # 分红配送
skill(name="roce-calculator")       # ROCE 计算
skill(name="market-analyzer")       # 市场分析
skill(name="market-systemic-risk")  # 市场系统性风险分析
skill(name="industry-analysis")     # 行业分析（排行/资金流/估值/轮动）
skill(name="shareholder-deep")      # 股东深度分析
skill(name="pdf-converter")        # PDF 转换
skill(name="email-sender")         # 邮件发送
skill(name="akshare-docs")        # AKshare API 文档查询
skill(name="risk-analysis")       # 综合风控（新闻+分位数）
skill(name="web-search")          # 网络实时搜索
skill(name="valuation-anchor")    # 估值锚点分析
skill(name="cninfo-search")       # 巨潮资讯网公告搜索
skill(name="chronos-timeline")    # CHRONOS 事件追踪分析
```

### 命令行运行方式（全部迁移到 core/src/skills/）
```bash
# 单独使用各 Skill
python .opencode/skills/core/src/skills/stock-analyzer/main.py 600519
python .opencode/skills/core/src/skills/technical-analyzer/main.py 600519
python .opencode/skills/core/src/skills/a-dividend-analyzer/main.py 600519
python .opencode/skills/core/src/skills/roce-calculator/main.py 600519
python .opencode/skills/core/src/skills/market-analyzer/main.py
python .opencode/skills/core/src/skills/market-systemic-risk/main.py
python .opencode/skills/core/src/skills/industry-analysis/main.py
python .opencode/skills/core/src/skills/shareholder-deep/main.py 000651
python .opencode/skills/core/src/skills/email-sender/main.py "收件人" "主题" "内容"
python .opencode/skills/core/src/skills/pdf-converter/main.py "file.pdf"
python .opencode/skills/core/src/skills/akshare-docs/main.py "stock_zh_a_spot"
python .opencode/skills/core/src/skills/web-search/main.py "查询内容"
python .opencode/skills/core/src/skills/risk-analysis/main.py 600519
python .opencode/skills/core/src/skills/valuation-anchor/main.py 600519
python .opencode/skills/core/src/skills/cninfo-search/main.py 600519
python .opencode/skills/core/src/skills/chronos-timeline/main.py 600519 回购
python .opencode/skills/core/src/skills/chronos-timeline/main.py 600519 利润分配 365
python .opencode/skills/core/src/skills/chronos-timeline/main.py 600338 锂矿 --relax --export-candidates cand.json   # 宽松匹配导出候选，供 AI 标注
python .opencode/skills/core/src/skills/chronos-timeline/main.py --build-report annotated.json               # 从 AI 标注构建 HTML 报告
```

## Architecture

### 分层架构

| 层级 | 目录 | 职责 |
|------|------|------|
| **向后兼容层** | `core/__init__.py` | 26 个包装函数 + 9 个类导出，委托给下层 Analyzer 类 |
| **分析器层** | `core/src/analyzers/` | 8 个 Analyzer 类，数据获取 + 计算逻辑 |
| **基础设施层** | `core/src/infra/` | CacheManager + ReportGenerator |
| **入口层** | `core/src/skills/` | 14 个 skill 的 `main.py`，参数解析 + 格式化输出 |

### 目录结构
```
stock-analyzer-skills_tushare/           # 项目根目录
├── .opencode/
│   └── skills/
│       ├── core/                        # 向后兼容导出层（委托给 src/）
│       │   └── __init__.py              #   26 个包装函数 + 类导出
│       │   └── src/                     # 核心源代码目录
│       │       ├── __init__.py
│       │       ├── config/
│   │       │   └── .env             # ⚠️ 统一配置文件（Tushare Token/SMTP/Tavily API Key）
│       │       ├── analyzers/           # 分析器层，9 个 Analyzer 类
│       │       │   ├── __init__.py
│       │       │   ├── market.py        # MarketAnalyzer（市场分析）
│       │       │   ├── technical.py     # TechnicalAnalyzer（技术分析）
│       │       │   ├── news.py          # NewsRiskAnalyzer（新闻风险）
│       │       │   ├── dividend.py      # DividendAnalyzer（分红配送）
│       │       │   ├── financial.py     # FinancialAnalyzer（财务健康 + ROCE）
│       │       │   ├── stock.py         # StockAnalyzer（个股估值 + 估值锚点）
│       │       │   ├── shareholder.py   # ShareholderAnalyzer（股东分析）
│       │       │   └── etf.py           # NationalTeamFundTracker（国家队ETF）
│       │       ├── infra/               # 基础设施层
│       │       │   ├── __init__.py
│       │       │   ├── cache.py         # CacheManager（缓存）
│       │       │   └── report.py        # ReportGenerator（评分 + 报告导出）
│       │       └── skills/              # 19 个 Skill 入口（薄封装层）
│       │           ├── stock-analyzer/main.py
│       │           ├── technical-analyzer/main.py
│       │           ├── a-dividend-analyzer/main.py
│       │           ├── buffett-checklist/main.py
│       │           ├── roce-calculator/main.py
│       │           ├── market-analyzer/main.py
│       │           ├── percentile-analyzer/main.py
│       │           ├── risk-analysis/main.py
│       │           ├── shareholder-deep/main.py
│       │           ├── valuation-anchor/main.py
│       │           ├── email-sender/main.py
│       │           ├── pdf-converter/main.py
│       │           ├── akshare-docs/main.py
│       │           ├── market-systemic-risk/main.py
│       │           ├── industry-analysis/main.py
│       │           ├── national-team-fund-tracker/main.py
│       │           ├── web-search/main.py
│       │           ├── cninfo-search/main.py
│       │           └── chronos-timeline/main.py
│       └── [skill-name]/             # 各 Skill 目录（SKILL.md + 旧入口）
│           ├── SKILL.md
│           └── scripts/              # 已迁移到 core/src/skills/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jinhongzou/akstock-analyzer-skills-opencode](https://github.com/jinhongzou/akstock-analyzer-skills-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
