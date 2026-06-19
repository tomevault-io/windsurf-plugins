---
trigger: always_on
description: 美股港股全栈数据工具包 — 覆盖行情(新浪+腾讯+东财push2)、K线(新浪+Yahoo)、技术指标(MA/MACD/RSI/KDJ/布林带)、基本面(东财datacenter三表+GMAININDICATOR+Yahoo quoteSummary+SEC XBRL)、资金面(东财push2his日级资金流)、期权(Yahoo)、SEC Filing(EDGAR)、搜索与工具(东财search+Yahoo+SEC CIK+全市场列表)八层数据源，内嵌全部调用代码，自包含零依赖外部文件。适用于美股港股个股分析、全市场筛选、财报解读、期权策略、SEC文件检索、资金流追踪、机构持仓分析等场景。
---


> 📦 项目主页：https://github.com/simonlin1212/global-stock-data — 更新、反馈、支持作者
> 
> 作者：Simon 林 · 抖音「Simon林」· 公众号「硅基世纪」

# 美股港股全栈数据工具包 V1.0

八层数据架构，18 个端点，5 个数据源，全部零鉴权，实测可用（2026-05-20 验证）。

**使用方式：** 将本文件放入 `~/.claude/skills/global-stock-data/SKILL.md`，Claude Code 会自动识别并在美股/港股相关对话中激活。

```
行情层（实时/延时）
├── 新浪财经     → 美股 gb_XXXX 36字段 / 港股 rt_hkXXXXX 25字段
├── 腾讯财经     → 美股 usXXXX 71字段 / 港股 r_hkXXXXX 78字段
└── 东财 push2   → 美股/港股 secid 实时行情，含中文名/涨跌幅/换手率

K线层（日/周/月/分钟）
├── 新浪          → 美股日K (回溯至1984年)
└── Yahoo chart   → 美股+港股 (v8 API, 零crumb)

技术指标层（纯计算，零额外依赖）
└── MA/EMA + MACD + RSI + KDJ + 布林带    基于K线OHLCV，纯Python计算

基本面层
├── 东财 datacenter → 美股/港股三表(资产负债+利润+现金流) + GMAININDICATOR(关键指标)
├── Yahoo crumb     → 23个模块(财务数据+关键指标+分析师+机构持仓)
└── SEC EDGAR XBRL  → 美股503个GAAP指标 (仅美股)

资金面层
└── 东财 push2his → 日级资金流(主力/大单/中单/小单) 美股+港股

期权层（仅美股）
└── Yahoo crumb → 期权链(calls+puts, 所有到期日) 仅美股(港股期权不在Yahoo覆盖范围)

SEC Filing层（仅美股）
├── EDGAR submissions → 10-K/10-Q/8-K 完整Filing列表
└── EDGAR XBRL        → 结构化财务指标(营收/净利/EPS等)

工具层
├── 东财 search    → 股票搜索(中英文, 含市场代码映射)
├── 东财 push2     → 全市场股票列表(涨跌幅/成交量排名, 美股5925只+港股18000+只)
├── Yahoo search   → 新闻资讯(按股票代码)
└── SEC CIK mapping → ticker↔CIK 映射 (仅美股)
```

## When to Activate

- 用户要查**美股/港股**行情（价格/涨跌幅/成交量）
- 用户要拉 K 线（日线/周线/月线/分钟线）
- 用户要看**财报**（资产负债表/利润表/现金流量表）
- 用户要看**关键财务指标**（PE/PB/ROE/利润率/目标价）
- 用户要看**分析师预期**（EPS预测/评级/目标价区间）
- 用户要看**机构持仓**（前十大机构/持股比例）
- 用户要看**资金流向**（主力/大单/中单/小单净流入）
- 用户要查**期权链**（calls/puts/到期日/Greeks）
- 用户要查 **SEC Filing**（10-K/10-Q/8-K/年报/季报）
- 用户要做**美股财报量化分析**（从 XBRL 拉多年营收/净利/EPS 趋势）
- 用户要**搜索股票**（中英文均可）
- 用户要看**美股/港股新闻**
- 用户要看**全市场涨跌幅排名**（当日涨幅/跌幅最大的股票）
- 用户要做**全市场筛选**（遍历美股/港股列表做初筛）
- 用户要看**关键财务指标概览**（营收/净利/EPS/ROE/ROA/资产负债率 中文版）
- 用户要看**技术指标**（MACD/RSI/KDJ/布林带/均线）
- 用户要判断**金叉死叉/超买超卖/变盘信号**
- 关键词：美股、港股、AAPL、苹果、腾讯、00700、TSLA、特斯拉、BABA、阿里巴巴、行情、K线、财报、PE、PB、ROE、分析师、目标价、期权、call、put、SEC、10-K、年报、季报、资金流、主力、机构持仓、新闻、涨幅排名、全市场、筛选、关键指标、MACD、RSI、KDJ、布林带、均线、金叉、死叉、超买、超卖、技术分析

---

## Prerequisites

```bash
pip install requests
```

| 依赖 | 版本要求 | 用途 |
|------|---------|------|
| requests | any | 所有 HTTP API 直连 |

> **极简依赖：** 仅需 requests，所有数据源均为直连 HTTP API，零第三方数据封装。

---

## 市场代码规则

### 东财 secid 前缀（push2/push2his 用）

| 前缀 | 市场 | 示例 |
|------|------|------|
| 105 | 美股 NASDAQ | `105.AAPL`, `105.TSLA` |
| 106 | 美股 NYSE | `106.BABA`, `106.JD` |
| 107 | 美股 ETF/其他 | `107.CRSH` |
| 116 | 港股 | `116.00700`, `116.09988` |

> **如何判断 105/106/107？** 调 `stock_search()` 获取 `MktNum` 字段自动映射。

### Yahoo Finance 代码格式

| 市场 | 格式 | 示例 |
|------|------|------|
| 美股 | 直接 ticker | `AAPL`, `TSLA`, `BABA` |
| 港股 | 四/五位数字 + `.HK` | `0700.HK`, `9988.HK` |

### 东财 datacenter SECUCODE 格式

| 市场 | 格式 | 示例 |
|------|------|------|
| 美股 NASDAQ | `TICKER.O` | `AAPL.O`, `TSLA.O` |
| 美股 NYSE | `TICKER.N` | `BABA.N`, `JD.N` |
| 港股 | `CODE.HK` | `00700.HK`, `09988.HK` |

---

## 共用 Helper 函数

### Yahoo Finance crumb 管理器

Yahoo quoteSummary/options 等 v7/v10 接口需要 cookie+crumb。以下 helper 自动获取并缓存：

```python
import requests

_yahoo_session = None

def get_yahoo_session() -> requests.Session:
    """获取带 crumb 的 Yahoo Finance session（自动缓存）"""
    global _yahoo_session
    if _yahoo_session and hasattr(_yahoo_session, '_crumb'):
        return _yahoo_session
    
    s = requests.Session()
    s.headers['User-Agent'] = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
    
    # Step 1: 获取 cookie
    s.get('https://fc.yahoo.com', timeout=10)
    
    # Step 2: 获取 crumb
    r = s.get('https://query2.finance.yahoo.com/v1/test/getcrumb', timeout=10)
    r.raise_for_status()
    s._crumb = r.text
    
    _yahoo_session = s
    return s

def yahoo_quote_summary(symbol: str, modules: list[str]) -> dict:
    """Yahoo quoteSummary 统一查询"""
    s = get_yahoo_session()
    r = s.get(f'https://query2.finance.yahoo.com/v10/finance/quoteSummary/{symbol}', params={
        'modules': ','.join(modules),
        'crumb': s._crumb,
    }, timeout=15)
    r.raise_for_status()
    results = r.json().get('quoteSummary', {}).get('result', [{}])
    return results[0] if results else {}
```

### 东财数据中心统一查询

```python
UA = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
DATACENTER_URL = "https://datacenter-web.eastmoney.com/api/data/v1/get"

def eastmoney_datacenter(report_name: str, columns: str = "ALL",
                          filter_str: str = "", page_size: int = 50,
                          sort_columns: str = "", sort_types: str = "-1") -> list[dict]:
    """东财数据中心统一查询"""
    params = {
        "reportName": report_name, "columns": columns,
        "filter": filter_str, "pageNumber": "1", "pageSize": str(page_size),
        "sortColumns": sort_columns, "sortTypes": sort_types,
        "source": "WEB", "client": "WEB",
    }
    r = requests.get(DATACENTER_URL, params=params, headers={"User-Agent": UA}, timeout=15)
    d = r.json()
    if d.get("result") and d["result"].get("data"):
        return d["result"]["data"]
    return []
```

---

## Layer 1: 行情层

### 1.1 美股实时行情 — 新浪 + 腾讯

两个独立数据源，任一可用即可。新浪字段侧重价格成交，腾讯字段更全（含52周高低/市值/PE）。

```python
import requests, re


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simonlin1212/global-stock-data](https://github.com/simonlin1212/global-stock-data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
