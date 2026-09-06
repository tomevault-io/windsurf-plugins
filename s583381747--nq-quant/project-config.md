---
trigger: always_on
description: Quantitative system for NQ (Nasdaq 100 E-mini Futures) targeting Prop Firm accounts.
---

# NQ Quantitative Trading System — CLAUDE.md

## Project Overview
Quantitative system for NQ (Nasdaq 100 E-mini Futures) targeting Prop Firm accounts.
Strategy is based on Lanto's ICT-derivative methodology: FVG as magnet/support/resistance,
multi-timeframe bias, displacement-based entries, and strict risk management.

---

## 1. INSTRUMENT & CONTRACT SPECS

- Instrument: NQ (Nasdaq 100 E-mini Futures)
- Tick size: 0.25 points
- Tick value: $5.00
- Point value: $20.00
- Commission: ~$2.05/side per contract (mini), ~$0.62/side (micro)
- Micro NQ (MNQ): 1/10th of NQ, point value = $2.00

---

## 2. SESSION DEFINITIONS (All times EST/ET)

| Session  | Start     | End       |
|----------|-----------|-----------|
| Asia     | 6:00 PM   | 3:00 AM   |
| London   | 3:00 AM   | 9:30 AM   |
| New York | 9:30 AM   | 4:00 PM   |

- **Opening Range Observation**: 9:30 AM – 10:00 AM EST → NO TRADES, observation only (cash market opens at 9:30)
- **Primary trading session**: NY (after 10:00 AM)
- Asia/London sessions: only trade "if up a lot", otherwise observe only
- **Daylight Saving Time**: Handle EST ↔ EDT transitions carefully in data pipeline

---

## 3. BIAS DETERMINATION

### 3.1 HTF FVG Analysis (Priority: Daily > 4H > 1H > 15m)
- Check for active HTF FVGs on D, 4H, 1H timeframes
- FVG = magnet/support/resistance; price is attracted toward unfilled gaps
- Higher TF FVGs carry more weight, UNLESS the HTF FVG is low quality (small, wicky)

### 3.2 HTF FVG State Machine
Each HTF FVG maintains a status:
- `untested` → price has not reached the FVG yet (acts as magnet/target)
- `tested_rejected` → price entered FVG but closed back out with displacement (continuation signal)
- `invalidated` → price **closed** through the FVG (reversal, seek next FVG/liquidity level)

### 3.3 Overnight/Session Level Bias
- Mark Asia session high/low as liquidity levels
- Mark London session high/low as liquidity levels
- Mark overnight high/low
- Opening range (9:00-10:00) high/low
- If price approaches overnight high but fails → short bias
- If price approaches overnight low but fails → long bias
- Observe price reaction at these levels to confirm or flip bias

---

## 4. ENTRY RULES

### 4.1 Core Setup: FVG Test + Rejection
1. Price approaches a valid FVG on 1m/5m timeframe
2. **Rejection candle**: candle tests the FVG, then closes on the correct side
   - Long: close ABOVE the FVG
   - Short: close BELOW the FVG
   - Candle must show displacement (not a doji, not wicky)
3. Entry on the close of the rejection candle

### 4.2 Reversal Entry
- Price sweeps Asia/London high or low
- Post-sweep: price shows displacement in opposite direction (long candles, no chop)
- A small FVG forms in the reversal direction → enter on test of this FVG

### 4.3 Continuation Entry
- Price tests a known FVG and gets rejected
- FVG status becomes `tested_rejected`
- Enter in the direction of the original trend

### 4.4 Uncertain/Retest Entry
- If reversal is NOT fluent, wait for price to retest the FVG
- Confirmation: retest + close on correct side → then enter

### 4.5 Entry Timeframes
- All entries on 1m or 5m charts
- HTF (1H, 4H, D) for bias only, never for entry timing

---

## 5. FVG DEFINITION & QUALITY

### 5.1 What is an FVG
Three consecutive candles where candle 1's high/low and candle 3's high/low do not overlap,
creating a gap at candle 2. Bullish FVG: candle 1 high < candle 3 low. Bearish FVG: candle 1 low > candle 3 high.

### 5.2 FVG Quality Criteria
- **Displacement**: the FVG-creating candle must be large relative to recent price action
- **Liquidity sweep**: FVG should form after taking out internal or external liquidity
- **Fluency**: recent candles leading into the FVG should be directional, not choppy
- **Wick ratio**: candles forming the FVG should have high body-to-range ratio (minimal wicks)
- Higher TF FVGs are inherently higher quality

### 5.3 FVG Invalidation
- Price **closes** through the FVG → FVG is invalidated
- An invalidated FVG can become an **inversion FVG** (former support becomes resistance, vice versa)

---

## 6. DISPLACEMENT & FLUENCY (Quantization)

### 6.1 Displacement — ⚠️ REQUIRES TUNING
A candle shows "displacement" when it demonstrates conviction and directional commitment.
**Starting quantitative definition (tune in Phase 1):**
- Candle body (abs(close - open)) > `DISPLACEMENT_ATR_MULT` × ATR(14)
- Candle body/range ratio > `DISPLACEMENT_BODY_RATIO`
- Candle engulfs at least 1 prior candle

| Parameter                | Starting Value | Tune Range  |
|--------------------------|---------------|-------------|
| DISPLACEMENT_ATR_MULT    | 0.8           | 0.5 – 1.5   |
| DISPLACEMENT_BODY_RATIO  | 0.60          | 0.50 – 0.80 |

### 6.2 Fluency Score — ⚠️ REQUIRES TUNING
Measures how directional/clean recent price action is (opposite of "chop").
Computed over a rolling window of N candles.

**Components:**
1. `directional_ratio`: fraction of candles moving in the same direction (e.g., 4/5 bullish = 0.8)
2. `avg_body_ratio`: mean(body/range) across window — high = clean candles, low = wicky
3. `avg_bar_size_vs_atr`: mean(range/ATR) — large bars = conviction

**Composite:** `fluency_score = w1 * directional_ratio + w2 * avg_body_ratio + w3 * avg_bar_size_vs_atr`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s583381747/nq-quant](https://github.com/s583381747/nq-quant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
