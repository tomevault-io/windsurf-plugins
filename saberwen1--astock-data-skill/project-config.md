---
trigger: always_on
description: A股量化数据分析工具，基于AkShare库获取A股行情、财务数据、板块信息等。用于回答关于A股股票查询、行情数据、财务分析、选股等问题。
---


# A股量化 - AkShare 数据接口

## 快速开始

安装依赖：
```bash
pip install akshare
```


# AKShare 股票数据接口

本文档为 AKShare 股票数据接口的渐进式披露文档，按接口功能分类组织。

---

## A股

### 股票市场总貌

#### 上海证券交易所

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 上海证券交易所-股票数据总貌 | 上海证券交易所-股票数据总貌 | [stock_sse_summary.md](./stock_metadata/stock_sse_summary.md) |

#### 深圳证券交易所

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 深圳证券交易所-市场总貌-证券类别统计 | 深圳证券交易所-市场总貌-证券类别统计 | [stock_szse_summary.md](./stock_metadata/stock_szse_summary.md) |
| 深圳证券交易所-市场总貌-地区交易排序 | 深圳证券交易所-市场总貌-地区交易排序 | [stock_szse_area_summary.md](./stock_metadata/stock_szse_area_summary.md) |
| 深圳证券交易所-股票行业成交 | 深圳证券交易所-统计资料-股票行业成交数据 | [stock_szse_sector_summary.md](./stock_metadata/stock_szse_sector_summary.md) |

### 个股信息查询

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-个股-股票信息 | 东方财富-个股-股票信息 | [stock_individual_info_em.md](./stock_metadata/stock_individual_info_em.md) |
| 雪球财经-个股-公司概况 | 雪球财经-个股-公司概况-公司简介 | [stock_individual_basic_info_xq.md](./stock_metadata/stock_individual_basic_info_xq.md) |

### 行情报价

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-行情报价 | 东方财富-行情报价 | [stock_bid_ask_em.md](./stock_metadata/stock_bid_ask_em.md) |

### 实时行情数据

#### 实时行情数据-东财

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-沪深京A股-实时行情数据 | 东方财富网-沪深京A股-实时行情数据 | [stock_zh_a_spot_em.md](./stock_metadata/stock_zh_a_spot_em.md) |
| 东方财富网-沪A股-实时行情数据 | 东方财富网-沪A股-实时行情数据 | [stock_sh_a_spot_em.md](./stock_metadata/stock_sh_a_spot_em.md) |
| 东方财富网-深A股-实时行情数据 | 东方财富网-深A股-实时行情数据 | [stock_sz_a_spot_em.md](./stock_metadata/stock_sz_a_spot_em.md) |
| 东方财富网-京A股-实时行情数据 | 东方财富网-京A股-实时行情数据 | [stock_bj_a_spot_em.md](./stock_metadata/stock_bj_a_spot_em.md) |
| 东方财富网-新股-实时行情数据 | 东方财富网-新股-实时行情数据 | [stock_new_a_spot_em.md](./stock_metadata/stock_new_a_spot_em.md) |
| 东方财富网-创业板-实时行情 | 东方财富网-创业板-实时行情 | [stock_cy_a_spot_em.md](./stock_metadata/stock_cy_a_spot_em.md) |
| 东方财富网-科创板-实时行情 | 东方财富网-科创板-实时行情 | [stock_kc_a_spot_em.md](./stock_metadata/stock_kc_a_spot_em.md) |
| 东方财富网-AB股比价 | 东方财富网-行情中心-沪深京个股-AB股比价-全部AB股比价 | [stock_zh_ab_comparison_em.md](./stock_metadata/stock_zh_ab_comparison_em.md) |

#### 实时行情数据-新浪

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-沪深京A股数据 | 新浪财经-沪深京A股数据,重复运行本函数会被新浪暂时封IP | [stock_zh_a_spot.md](./stock_metadata/stock_zh_a_spot.md) |

#### 实时行情数据-雪球

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 雪球-行情中心-个股 | 雪球-行情中心-个股 | [stock_individual_spot_xq.md](./stock_metadata/stock_individual_spot_xq.md) |

### 历史行情数据

#### 历史行情数据-东财

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-沪深京A股日频率数据 | 东方财富-沪深京A股日频率数据;历史数据按日频率更新,当日收盘价请在收盘后获取 | [stock_zh_a_hist.md](./stock_metadata/stock_zh_a_hist.md) |

#### 历史行情数据-新浪

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-沪深京A股的数据 | 新浪财经-沪深京A股的数据,历史数据按日频率更新 | [stock_zh_a_daily.md](./stock_metadata/stock_zh_a_daily.md) |

#### 历史行情数据-腾讯

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 腾讯证券-日频-股票历史数据 | 腾讯证券-日频-股票历史数据;历史数据按日频率更新,当日收盘价请在收盘后获取 | [stock_zh_a_hist_tx.md](./stock_metadata/stock_zh_a_hist_tx.md) |

### 分时数据

#### 分时数据-新浪

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-沪深京A股分时数据 | 新浪财经-沪深京A股股票或者指数的分时数据,目前可以获取1,5,15,30,60分钟的数据频率 | [stock_zh_a_minute.md](./stock_metadata/stock_zh_a_minute.md) |

#### 分时数据-东财

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-沪深京A股-每日分时行情 | 东方财富网-行情首页-沪深京A股-每日分时行情;该接口只能获取近期的分时数据 | [stock_zh_a_hist_min_em.md](./stock_metadata/stock_zh_a_hist_min_em.md) |

### 日内分时数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-分时数据 | 东方财富-分时数据 | [stock_intraday_em.md](./stock_metadata/stock_intraday_em.md) |
| 新浪财经-日内分时数据 | 新浪财经-日内分时数据;只能获取近期的数据,此处仅返回大单数据 | [stock_intraday_sina.md](./stock_metadata/stock_intraday_sina.md) |
| 东方财富-股票行情-盘前数据 | 东方财富-股票行情-盘前数据 | [stock_zh_a_hist_pre_min_em.md](./stock_metadata/stock_zh_a_hist_pre_min_em.md) |

### 历史分笔数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 腾讯财经-历史分笔数据 | 每个交易日16:00提供当日数据 | [stock_zh_a_tick_tx.md](./stock_metadata/stock_zh_a_tick_tx.md) |

### 同行比较

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-行情中心-同行比较-成长性比较 | 东方财富-行情中心-同行比较-成长性比较 | [stock_zh_growth_comparison_em.md](./stock_metadata/stock_zh_growth_comparison_em.md) |
| 东方财富-行情中心-同行比较-估值比较 | 东方财富-行情中心-同行比较-估值比较 | [stock_zh_valuation_comparison_em.md](./stock_metadata/stock_zh_valuation_comparison_em.md) |
| 东方财富-行情中心-同行比较-杜邦分析比较 | 东方财富-行情中心-同行比较-杜邦分析比较 | [stock_zh_dupont_comparison_em.md](./stock_metadata/stock_zh_dupont_comparison_em.md) |
| 东方财富-行情中心-同行比较-公司规模 | 东方财富-行情中心-同行比较-公司规模 | [stock_zh_scale_comparison_em.md](./stock_metadata/stock_zh_scale_comparison_em.md) |

---

## A股-CDR

### 历史行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 上海证券交易所-科创板-CDR | 上海证券交易所-科创板-CDR | [stock_zh_a_cdr_daily.md](./stock_metadata/stock_zh_a_cdr_daily.md) |

---

## B股

### 实时行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-实时行情数据 | 东方财富网-实时行情数据 | [stock_zh_b_spot_em.md](./stock_metadata/stock_zh_b_spot_em.md) |
| B股数据-新浪财经 | B股数据是从新浪财经获取的数据 | [stock_zh_b_spot.md](./stock_metadata/stock_zh_b_spot.md) |

### 历史行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saberwen1/astock-data-skill](https://github.com/saberwen1/astock-data-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
