---
trigger: always_on
description: 多币种裸K突破策略·激进版（v1.2.0）。每15分钟扫描10个精选主流币+平台币，基于300根K线识别结构、构建箱体、评估质量，执行激进突破回踩交易，支持动态仓位和全持仓管理。【第三方策略，非 OKX 官方出品】
---


# 多币种裸K突破策略·激进版 V1.2.0（第三方 · 非 OKX 官方）

> ⚠️ **声明**：本策略为第三方社区作品，**不代表 OKX 官方立场或产品**，仅通过 OKX CLI 访问 OKX 交易所 API。
>
> 扫描 10 个精选主流币 + 平台币，每 15 分钟触发一次，基于 300K 结构识别箱体，执行激进突破回踩交易。

## 核心定位（激进 vs 稳健对比）

| 维度 | BTC裸K（稳健版） | 本策略（激进版） |
|------|----------------|----------------|
| 触发频率 | 5 分钟 | 15 分钟 |
| 交易对数量 | 1（仅 BTC） | 10 精选币种 |
| 每币种最大持仓 | 1 | 1（全币种并行） |
| 结构评分要求 | ≥ 中（跳过低） | ≥ 低即可交易 |
| 盈亏比要求 | ≥ 1:2 | ≥ 1:1.5（激进） |
| 频率限制 | 每小时1笔，每日5笔 | 每小时2笔，每日8笔 |
| 最大并发持仓数 | 1 | 5 |
| 回撤熔断 | 5% 日回撤停止 | 8% 日回撤停止 |
| 止损方式 | 固定止损 | 激进移动止损 |

---

## 支持交易对（10个精选）

### 主流币（8个）
```
BTC-USDT-SWAP  ETH-USDT-SWAP  SOL-USDT-SWAP  BNB-USDT-SWAP
XRP-USDT-SWAP  ADA-USDT-SWAP  DOGE-USDT-SWAP  AVAX-USDT-SWAP
```

### 平台币（2个）
```
LINK-USDT-SWAP  HYPE-USDT-SWAP
```

---

## 依赖 Skills（必须先安装）

- `okx-cex-market` — 获取 K线、实时价格、ATR、技术指标
- `okx-cex-trade` — 执行买入和卖出订单、设置止盈止损
- `okx-cex-portfolio` — 查询账户持仓、余额、净值
- `qclaw-cron-skill` — 注册和管理 15 分钟定时任务

---

## 参数说明

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `risk_per_trade` | number | ❌ | 1.5 | 单笔风险占账户净值百分比（激进默认值 1.5%） |
| `max_daily_trades` | integer | ❌ | 8 | 每日最大交易笔数 |
| `max_hourly_trades` | integer | ❌ | 2 | 每小时最大交易笔数 |
| `max_concurrent_positions` | integer | ❌ | 5 | 最大并发持仓数数 |
| `daily_drawdown_limit` | number | ❌ | 8 | 日回撤熔断阈值（%），超过停止开仓 |
| `leverage` | integer | ❌ | 10 | 默认杠杆倍数（10x 激进）
| `atr_stop_multi` | number | | 1.2 | ATR止损倍数（优化：0.8->1.2，减少噪音触发） |
| `min_atr_filter` | number | | 0.0005 | 最小ATR过滤（USDT值），低于此值跳过交易 |
 |
| `profile` | string | ❌ | demo | 实盘 live / 模拟盘 demo |
| `pairs` | string[] | ❌ | 全量10 | 指定扫描的交易对列表 |
| `scan_mode` | string | ❌ | all | 扫描模式：`all` 全量 / `focus` 仅重点关注 |

**重点关注币种（focus 模式）**：
`BTC ETH SOL BNB XRP ADA DOGE AVAX LINK HYPE`

---

## 执行流程总览

```
每 15 分钟触发
    │
    ▼
① 采集行情数据（300根1H + 300根15M）
    │
    ▼
② 并行扫描所有交易对
    │
    ├── 每对独立执行 Step 3-7
    │
    ▼
③ 结构分析（300K结构）
    │
    ▼
④ 市场状态识别
    │
    ▼
⑤ AI 综合决策
    │
    ▼
⑥ 频率 & 风控检查
    │
    ▼
⑦ 执行下单 + 设置止损
    │
    ▼
⑧ 持仓监控 + 激进移动止损
```

---

## Step 1 · 行情数据采集（每币种独立）

对每个候选交易对并行执行：

```bash
# 获取 300 根 1H K线（结构分析用）
okx market candles <instId> --bar 1H --limit 300

# 获取 300 根 15M K线（入场确认用）
okx market candles <instId> --bar 15m --limit 300

# 获取当前价格
okx market ticker <instId>

# 获取 ATR(14)
okx market indicator atr <instId> --bar 1H --params 14

# 获取 ATR(14) 15M
okx market indicator atr <instId> --bar 15m --params 14
```

---

## Step 2 · 结构分析（300K 核心）

对每个币种 AI 自动完成：

### ① 提取 Swing High / Swing Low

从 300 根 1H K线中识别局部高点和低点，过滤噪音波动。

### ② 构建 1H 箱体

判断是否满足箱体条件：
- 上沿至少被触及 ≥ 2 次
- 下沿至少被触及 ≥ 2 次
- 高度 ≥ ATR(14) × 1.5（保证最小波动空间）

**激进放宽**：若触及 ≥ 1 次 + 结构清晰（边界干净、无毛刺），仍可接受。

输出：
```
箱体上沿：<价格>
箱体下沿：<价格>
箱体高度：<价差>
结构评分：高/中/低
```

### ③ 箱体质量评分

| 评分 | 条件 | 仓位系数 |
|------|------|---------|
| 高 | 边界干净、触及≥3次、宽箱体 | ×1.0 |
| 中 | 边界较干净、触及2次 | ×0.7 |
| 低 | 边界模糊、触及1次（激进模式允许） | ×0.4 |

**激进策略特殊规则**：允许交易低评分结构，但需满足：
- 市场处于震荡（非趋势）
- 盈亏比 ≥ 1:2（用更严格盈亏比弥补结构不足）
- 箱体高度 ≥ ATR × 2

---

## Step 3 · 市场状态识别

基于 300 根 1H K线判断：

```
市场状态：上升趋势 / 下降趋势 / 震荡 / 高风险
```

**判断规则**：
- 高点持续抬高 + 低点持续抬高 → 上升趋势
- 高点持续降低 + 低点持续降低 → 下降趋势
- 高低点重叠交错 → 震荡
- 连续 ≥ 3 根强趋势K + ATR > 均值 1.5倍 → 高风险

**激进策略规则**：
- 上升趋势：只做多（顺趋势突破）
- 下降趋势：只做空（顺趋势突破）
- 震荡：多空均可
- 高风险：仅当盈亏比 ≥ 1:3 时允许做空（逆势需极高赔率）

---

## Step 4 · AI 综合决策（激进版）

### ① 交易信号识别

**突破类型判定**：

| 突破类型 | 条件 | 操作 |
|---------|------|------|
| 真突破（强烈） | 收盘突破 + 实体明显 + 回踩触边界 | 立即入场 |
| 真突破（保守） | 突破后小幅回抽（≤ ATR/2） | 等待回踩确认后入场 |
| 假突破 | 立即回到箱体内部 / 影线突破 | 跳过 |

**激进特殊**：若 15M 出现强趋势信号（连续 3 根同色 K + 放量），允许影线突破后直接入场。

### ② 方向确定

- 上破箱体上沿 → 做多
- 下破箱体下沿 → 做空
- 趋势方向 + 突破同向 → 优先交易
- 趋势方向 + 突破反向 → 仅结构评分高时交易

### ③ 盈亏比评估

```
风险距离 = 入场价 - 止损价
目标距离 = 止盈价 - 入场价
盈亏比 = 目标距离 / 风险距离
```

| 结构评分 | 最低盈亏比要求 |
|---------|--------------|
| 高 | ≥ 1:1.5 |
| 中 | ≥ 1:2.0 |
| 低 | ≥ 1:2.5 |

### ④ 动态仓位计算

```python
单笔风险金额 = 账户权益 × risk_per_trade / 100
仓位张数 = floor(单笔风险金额 × 杠杆 / (ATR_15m × ctVal))
实际仓位 = 仓位张数 × 仓位系数
```

**激进系数调整**：
- 结构评分 = 高 → 仓位 × 1.0
- 结构评分 = 中 → 仓位 × 0.7
- 结构评分 = 低 → 仓位 × 0.4
- ATR 偏高（> 均值 1.3x）→ 仓位 × 0.5（激进不降仓，减少交易频率）
- ATR 偏低（< 均值 0.7x）→ 仓位 × 1.2（激进模式允许超注）

### ⑤ 频率 & 风控检查

```id="limit"
全局限制（所有币种共享）：
- 每小时最多 2 笔
- 每日最多 8 笔
- 同时最多 5 个仓位
- 日回撤 ≥ 8% → 停止所有开仓

单币种限制：
- 每币种每小时最多 1 笔
- 每币种每日最多 2 笔
- 同方向 1 小时内只允许 1 次
```

### ⑥ 最终决策（必须输出）

```id="decision"
1. 开多 <instId>（理由 + 仓位张数 + 止损价 + 止盈价）
2. 开空 <instId>（理由 + 仓位张数 + 止损价 + 止盈价）
3. 本轮跳过（理由）
```

---

## Step 5 · 执行下单

### Step 5.1 前置检查

```bash
# 查询账户 USDT 余额
okx account balance USDT --profile <profile>

# 查询当前所有永续持仓
okx account positions --instType SWAP --profile <profile>

# 获取当前持仓数量（计算并发数）
```

检查：
1. 可用 USDT 余额充足
2. 当前并发持仓数 < max_concurrent_positions
3. 今日交易笔数 < max_daily_trades
4. 本小时交易笔数 < max_hourly_trades
5. 同币种本小时未开仓
6. 日回撤 < daily_drawdown_limit

### Step 5.2 执行市价开仓

```bash
# 做多（激进使用10x杠杆）
okx swap place --instId <instId> --side buy --ordType market \
  --sz <合约张数> --tdMode cross --posSide long \
  --lever <leverage> --tgtCcy quote_ccy \
  --tag agentTradeKit --profile <profile>

# 做空
okx swap place --instId <instId> --side sell --ordType market \
  --sz <合约张数> --tdMode cross --posSide short \
  --lever <leverage> --tgtCcy quote_ccy \
  --tag agentTradeKit --profile <profile>
```

记录：开仓价格（ticker last）、合约张数。

### Step 5.3 设置止损 & 止盈

**止损**（激进移动止损）：

```bash
# 多单止损（激进：止损距离开仓 1.2 ATR）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skynet2000/naked-k-multi-coin](https://github.com/skynet2000/naked-k-multi-coin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
