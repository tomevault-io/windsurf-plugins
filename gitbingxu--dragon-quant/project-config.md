---
trigger: always_on
description: > 龙头战法量化筛选系统 · AI Agent 操作手册
---

# AGENTS.md — dragon-quant

> 龙头战法量化筛选系统 · AI Agent 操作手册

---

## 这是什么

一套纯 Python 3 的 A 股龙头筛选系统。从当日涨停榜出发，评估涨停股的龙头质量并加权排名输出；同时支持日志查询、SQLite 持久化、龙头回测与 Web UI 可视化。

系统内置**两套评分体系**，分别由 `scan`（v1）与 `scan_v2`（v2）命令触发，并存互不影响：

- **v1（旧四维，默认）**：带动性 35% / 领涨性 25% / 抗跌性 15% / 资金承接 25%，简单加权求和。
- **v2（新五维「识别真龙」）**：带动性 30% / 领涨性 25% / 抗跌性 15% / 流动性 20% / 资金承接 10%，**门槛+加权两段式聚合**（四大特征任一低于门槛即一票否决，资金承接不否决仅加权贡献）。详见《评分器Refactor.md》。

数据源：同花顺（板块数据）+ 雪球（个股 K 线/分时）+ 腾讯（批量行情/收盘盘口），不依赖任何付费行情接口。东财 provider 仍保留但默认不参与扫描。

> **板块口径已切换为「行业板块」**（同花顺 `thshy`/`hyzjl`，约 90 个真实行业，code 为 881xxx），不再用概念板块（`gn`/`gnzjl`）。

---

## 快速使用

```bash
cd ~/repo/dragon-quant

# 批量扫描（默认 v1 评分器）
python -m dragon_quant
python -m dragon_quant scan --top 25 --candidates 5 --workers 2

# 使用 v2 五维「识别真龙」评分器
python -m dragon_quant scan_v2 --top 5

# 强制执行（跳过交易时段拦截 + DB 缓存）
python -m dragon_quant scan_v2 --force

# 概念板块黑名单管理（拉取领涨/领跌板块时过滤）
python -m dragon_quant blacklist list
python -m dragon_quant blacklist add "次新股"
python -m dragon_quant blacklist remove "次新股"

# 持久化数据管理
python -m dragon_quant storage status        # 查看存储状态
python -m dragon_quant storage size          # 磁盘占用
python -m dragon_quant storage clear --all   # 清理全部

# 按评分体系回测 / 查看 UI
python -m dragon_quant review --source v1 --date 20260519
python -m dragon_quant review --ui-only --source v2
```

### 前置条件

板块数据用**同花顺**，**无需 Cookie**（curl + GBK 直取，无 Playwright/反爬）。个股数据依赖雪球，需配雪球 Cookie：

```bash
# 查看状态
python -c "from dragon_quant.providers.cookie import get_xq; print(f'雪球: {bool(get_xq())}')"

# 手动设置雪球 Cookie（推荐）
python -m dragon_quant.providers.cookie set --cookie "xq_a_token=...; xq_is_login=1; u=..." --source xq

# 自动获取（需要 playwright）
python -m dragon_quant.providers.cookie fetch --source xq
```

Cookie 文件位置：
- 雪球：`~/Library/Application Support/dragon-quant/cookies/xueqiu`
- 东财（保留备用）：`~/Library/Application Support/dragon-quant/cookies/eastmoney`

---

## 架构总览

```
dragon_quant/
├── __init__.py / __main__.py    # 入口
├── cli.py                       # argparse CLI（scan/scan_v2/logs/data/review/vpa/storage/blacklist）
├── orchestrator.py              # 编排主流程 (Phase A→F)，含 v1/v2 双分支
├── data.py                      # 原子数据查询 API
├── rate_limit.py                # 分组并发调度器
├── analyze.py                   # 子进程打分入口（v1 路径，保留）
│
├── providers/                   # 数据源适配层
│   ├── base.py                  # StockProvider ABC + scorers_v2 新增板块 K 线方法
│   ├── ths.py                   # 同花顺 — 行业排行(curl)/成分股(HTML)/板块1分K/历史5分K
│   ├── eastmoney.py             # 东财 — 保留，默认不参与扫描
│   ├── xueqiu.py                # 雪球 — 个股日K/分时，需 Cookie
│   ├── tencent.py               # 腾讯 — 零认证，批量行情 + 收盘盘口(bid1)
│   ├── browser.py               # Playwright 浏览器会话（Cookie 获取/页面渲染）
│   └── cookie.py                # Cookie 管理 + CLI
│
├── scorers/                     # v1 四维评分器（旧，保留不动）
│   ├── drive.py / anti_drop.py / leadership.py / absorption.py
│
├── scorers_v2/                  # v2 五维「识别真龙」评分器（✅ 新增）
│   ├── base.py                  # DragonVerdict + 1分K对齐/归一化涨幅/排名分位工具
│   ├── registry.py              # 全部权重/门槛/阈值常量（集中调参）
│   ├── drive.py                 # 带动性 30%（封板最早/带动板块脉冲检测/板块共鸣）
│   ├── leadership.py            # 领涨性 25%（连板最多/5日涨幅板块内分位）
│   ├── anti_drop.py             # 抗跌性 15%（大盘+板块双基准横盘稳住/率先起飞）
│   ├── liquidity.py             # 流动性 20%（换手充沛度/封板质量，一字不罚）
│   ├── absorption.py            # 资金承接 10%（跨板块虹吸，回看10交易日）
│   └── aggregator.py            # 门槛+加权聚合 → DragonVerdict
│
├── vpa/                         # 量价分析（独立模块，插件式因子 FACTORS）
│   ├── engine.py / types.py / report.py
│   └── factors/                 # vol_amount / trend_verify / breakout / divergence
│
├── cache/data_cache.py          # 内存+本地双重缓存
├── logging/                     # 结构化日志 + 自然语言报告
│   ├── logger.py                # ScanLogger
│   └── reporter.py              # ReportBuilder（v1 四维 + v2 五维报告）
├── storage/                     # 统一持久化
│   ├── paths.py / manager.py
│   └── db.py                    # SQLite（v1/v2 分表 + vpa_analysis/sector_blacklist）
├── utils/trading.py            # 交易日历 + 涨停判断 + 买入日定位
├── review.py                    # 龙头回测验证
├── web_ui/                      # 回测结果 Web UI（Vite+React+TS+Mantine / stdlib HTTPServer）
└── models/types.py             # dataclass 数据模型
```

---

## 执行流程

编排器 `orchestrator.scan(scorers="v1"|"v2")` 分 6 个阶段。下表标注 v1/v2 差异：

| Phase | 做什么 | v1 | v2 |
|-------|--------|----|----|
| **A** 板块排行 | 同花顺·行业板块涨跌幅榜 | 领涨 Top8 + 领跌 Top20 | 领涨 **Top5** + 领跌 Top20；过滤 DB 黑名单 |
| **B** 候选筛选 | 每领涨板块取候选股，过滤 ST+双创+北交所 | 每板块按5日涨幅取前5 | 每板块**当日所有涨停股**(pct≥9.9) |
| **C** 连板+排序 | 雪球日K 算连板天数 → 按(连板,概念数)降序，**对候选池全部评分(不截断)** | — | 额外写 `Candidate.fived_pct`（5日涨幅） |
| **D** 并发加载 | 板块/个股 K 线 + 腾讯批量行情 | 板块当日5分K + 个股1分K | 板块**历史10日5分K** + 板块**当日1分K** + 大盘1分K + 全候选1分K（封板池）|
| **E** 打分 | 逐候选股评分（候选池全部，无 Top N 截断） | `_score_one`（四维加权）| `_score_one_v2`（五维门槛+加权 → DragonVerdict）|
| **F** 输出+持久化 | 排序 + 报告 + SQLite + 5日去重 | 四维报告 + 写 v1 分表 | 五维报告 + 写 v2 分表 |

总耗时约 40-80 秒（取决于网络、并发数、v2 拉取量更大）。

---

## 数据模型

核心类型全部在 `models/types.py`：

- **KBar** — 一根 K 线（timestamp, OHLCV, 涨跌幅, 换手率, 成交额）
- **StockInfo** — 股票基本信息 + 当日快照（含 `five_day_return`）
- **Quote** — 实时行情快照（现价/涨跌幅/换手率/市值/PE/量比…）+ **收盘盘口 `bid1_price`/`bid1_volume`/`ask1_volume`**（gtimg f[9]/f[10]/f[20]，单位手）
- **SectorPerformance** — 板块行情（代码/名称/涨跌幅/振幅）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gitBingxu/dragon-quant](https://github.com/gitBingxu/dragon-quant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
