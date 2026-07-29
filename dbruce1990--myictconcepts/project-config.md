---
trigger: always_on
description: I am developing an ICT (Inner Circle Trader) concepts indicator in Pine Script v6 that recreates the commercial TTrades Fractal Model with personal enhancements. The project is based on multi-timeframe analysis using swing points, Fair Value Gaps (FVGs), and Change in State of Delivery (CISD) detection in addition to TTrades' Daily Bias (or I call it Next Candle Model) for statistical validation. The goal is to create a reliable, non-repainting indicator that integrates seamlessly with the TTrad
---

# GitHub Copilot Repository Instructions

## Project Context
I am developing an ICT (Inner Circle Trader) concepts indicator in Pine Script v6 that recreates the commercial TTrades Fractal Model with personal enhancements. The project is based on multi-timeframe analysis using swing points, Fair Value Gaps (FVGs), and Change in State of Delivery (CISD) detection in addition to TTrades' Daily Bias (or I call it Next Candle Model) for statistical validation. The goal is to create a reliable, non-repainting indicator that integrates seamlessly with the TTrades methodology while providing a solid foundation for future enhancements. The project is structured to allow for easy reference to existing indicators and methodologies, with a focus on maintaining high code quality and comprehensive documentation. Every trade setup is based on the Next Candle Model, which is a statistical bias system with a 63-67% success rate. Once we identify a C2 (Candle 2) closure, we anticipate a C3 (Candle 3) continuation. We can also trade C4 (Candle 4) based on C3 closures as the point is to trade the next candle in a swing point formation (also known as a pivot point or williams fractal...simply a 3-candle pattern with a high/low in the middle). 

## Key Concepts & Definitions

### ICT Trading Terminology
- **Order Block**: The candle or series of candles that sweep liquidity (a swing high or low)
- **Swing Point**: A 3-candle pattern with a high/low in the middle that indicates a potential reversal point in the market
- **Protected Swing Point**: Derived from how TTrades defines OB (Order Block). It is a swing point that has swept another swing point's liquidity and formed an OB.
- **CISD (Change in State of Delivery)**: A candle closure beyond the lowest/highest opening price of thecandle or series of candles that swept liquidity. A CISD validates an order block and creates a protected swing point.
- **HTF (Higher Time Frame)**: The timeframe that provides bias and target levels for lower time frame execution. Any higher timeframe than the one we are looking at.
- **LTF (Lower Time Frame)**: Any timeframe lower than the current time frame we are discussing.
- **FVG (Fair Value Gap)**: 3-candle pattern with gap between high/low of outer candles. For example a bullish FVG is where there is no overlap between the high of the first candle and the low of the third candle. Reverse for bearish FVG.
- **Imbalance**: Represents an inefficiency in price delivery. There are different kinds of imbalances, such as FVG's and Volume Imbalances (VI's). There are also others such as New Day Opening Gaps (NDOG's) and New Week Opening Gaps (NWOG's), which are a kind of volume imbalance in my opinion. Any gap between candles is considered an imbalance.
- **BISI**: A type of FVG. BISI stands for Buyside Imbalance, Sellside Inefficiency.
- **SIBI**: A type of FVG. SIBI stands for Sellside Imbalance, Buyside Inefficiency.
- **C2 Closure**: A C2 closure is a candle that sweeps the high or low of the previous candle and closes back inside the previous candle's range. This is a critical part of the TTrades Fractal Model.
- **C3 Closure**: A specific pattern where a swing point formation (3-candle pattern) aligns with Next Candle Model logic. Occurs when: (1) A swing point forms (high[1] > high[2] AND high < high[1] for swing high, or low[1] < low[2] AND low > low[1] for swing low), AND (2) The current candle's close aligns with the swing direction (close < low[1] for bearish swing high, close > high[1] for bullish swing low). This creates a bias for C4 in the direction of the swing.
- **C4**: The fourth candle in a swing formation that can be traded based on the C3 closure. Requires C3 to close beyond C2's high/low as per the next candle model rules. 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbruce1990/MyICTConcepts](https://github.com/dbruce1990/MyICTConcepts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
