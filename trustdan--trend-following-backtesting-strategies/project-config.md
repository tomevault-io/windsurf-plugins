---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **trend-following strategy backtesting research project** that systematically tests 40+ Pine Script strategies across 21 securities (11 individual stocks + 10 sector ETFs). The project documents the journey from -93% to +40% through iterative learning and optimization, with 294 completed backtests.

**Key Finding:** Strategy performance is highly asset-dependent. What works on SPY may fail on other securities, requiring systematic sector-by-sector testing.

## Repository Structure

```
trend-following-backtesting-strategies/
├── pine-scripts/          # 48 TradingView Pine Script strategy files
│   └── [rank]_PF-[X.XXX]_[strategy-name].pine
├── sectors/               # 21 sector folders (11 stocks + 10 ETFs)
│   ├── technology-MSFT/
│   ├── healthcare-UNH/
│   ├── etfs-SPY/
│   └── ... (18 more)
│       ├── screenshots/   # Backtest result images
│       └── strategies/    # Tested strategy files
├── docs/                  # Testing guides and documentation
├── analysis.csv           # Performance data for all backtests
├── DISCOVERIES_AND_LEARNINGS.md  # Comprehensive findings (large file)
├── README.md              # Project overview and methodology
└── bug-log.txt            # Screenshot validation log
```

## Architecture & Data Flow

### 1. Strategy Development Workflow
- **Generation**: AI (ChatGPT/Claude) generates Pine Script strategies embodying specific trading hypotheses
- **Testing**: Manual testing in TradingView across 21 securities with standardized settings
- **Screenshot Capture**: Windows Snipping Tool (Win+Shift+S) captures Strategy Tester panel
- **Validation**: Visual inspection for "exit bugs" (purple arrows stopping mid-chart)
- **Documentation**: Results recorded in analysis.csv and DISCOVERIES_AND_LEARNINGS.md

### 2. File Naming Conventions

**Pine Scripts**: `[RANK]_PF-[X.XXX]_SPY_seykota_alt[N]_[description].pine`
- Example: `01_PF-1.474_SPY_seykota_alt31_fractional_breakeven.pine`
- RANK: Performance ranking (1 = best)
- PF: Profit Factor from SPY backtest
- alt[N]: Strategy variation number

**Screenshots**: `alt[X]-[result]_reduced.png`
- Example: `alt26-very-profitable_reduced.png`
- `_reduced`: Indicates 35% size reduction via resize_images.ps1

### 3. Data Structure

**analysis.csv format**:
```
Strategy,Security,Result_%,Profit_Factor,Win_Rate_%,Max_DD_%,Trades,Performance_Category,Options_Suitable
```

**Performance Categories**:
- Exceptional/Outstanding: PF > 2.5
- Very Profitable: PF 1.7-2.5
- Profitable: PF 1.2-1.7
- Scratch: PF 0.9-1.2
- Unprofitable: PF 0.5-0.9
- Catastrophic: PF < 0.5

## Common Development Tasks

### Testing New Strategies

When adding a new strategy for multi-sector testing:

1. **Copy strategy file** to `pine-scripts/` with proper naming
2. **Test on first security** in TradingView (recommended: QQQ or SPY)
3. **Capture screenshot** using Win+Shift+S workflow
4. **Validate screenshot** for exit bugs (see bug-log.txt criteria)
5. **If valid**, test across remaining 20 securities
6. **Save screenshots** to appropriate `sectors/[name]/screenshots/` folders
7. **Resize images** if needed: `.\resize_images.ps1`
8. **Record metrics** in analysis.csv
9. **Document findings** in DISCOVERIES_AND_LEARNINGS.md

### Screenshot Management

**Resize images to fit Claude Code's processing limits**:
```powershell
.\resize_images.ps1
```
- Reduces by 35% (to 65% of original)
- Recursively processes all PNG/JPG in sectors/
- Skips files already containing "_reduced"
- Deletes originals after successful resize

### Analyzing Results

**View top performers**:
```bash
head -30 analysis.csv
```

**Count strategies per security**:
```bash
grep "^Alt10," analysis.csv | wc -l
```

**Find best performing strategy for a security**:
```bash
grep ",MSFT," analysis.csv | sort -t, -k3 -rn | head -5
```

## Key Project Insights

### Critical Discovery: Asset-Specific Performance
- **Alt31** (best SPY strategy): +40.1%, PF 1.474
- **Same strategy on other assets**: Often fails
- **Lesson**: Must test strategies across all target securities

### Top Universal Strategies (Work Across Multiple Assets)
1. **Alt10** (Profit Targets): 76.19% success rate (16/21 profitable)
2. **Alt45** (Dual-Momentum): 66.67% success rate
3. **Alt46** (Sector-Adaptive): 61.90% success rate
4. **Alt43** (Volatility-Adaptive): 61.90% success rate

### Sector Insights from 294 Backtests
- **Healthcare**: 12/13 strategies profitable (best sector)
- **Utilities**: 0/14 strategies profitable (worst sector)
- **XLV** (Healthcare ETF): Alt46 +34.80% (best ETF result)
- **CAT** (Industrials): Alt47 3.96% max DD (lowest drawdown ever)

### Exit Bug Detection (Critical for Data Quality)
**Bug Pattern** (INVALID):
- Purple exit arrows stop partway through chart (especially after ~2017)
- Equity curve begins tracking price line closely
- Strategy accidentally became buy-and-hold
- Example: SPY Alt39 showed +121.89% but exits stopped after 2017

**Healthy Pattern** (VALID):
- Purple arrows continue throughout 2010-2025
- Equity curve diverges from price
- Consistent trading activity through recent periods


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trustdan/trend-following-backtesting-strategies](https://github.com/trustdan/trend-following-backtesting-strategies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
