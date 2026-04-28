---
trigger: always_on
description: Provides technical indicators, signal detection, backtesting, and optimization.
---

# CLAUDE.md

TrendCraft - Technical analysis library for TypeScript (pnpm workspace monorepo)

## Project Overview

A library for analyzing financial data (stocks, crypto, etc.).
Provides technical indicators, signal detection, backtesting, and optimization.
`@trendcraft/chart` provides a Canvas-based charting library with auto-detection of TrendCraft indicator series, framework bindings (React/Vue), and a headless API.

## Directory Structure

```
packages/
├── chart/              # npm package "@trendcraft/chart"
│   ├── src/
│   │   ├── core/           # Data layer, scales, layout, viewport, plugin types
│   │   ├── renderer/       # Canvas render pipeline, overlays, axis
│   │   ├── series/         # Series type renderers (candlestick, line, band, cloud, etc.)
│   │   └── integration/    # Series introspection, indicator presets, live feed
│   ├── react/              # React wrapper (TrendChart)
│   ├── vue/                # Vue wrapper (TrendChart)
│   └── examples/           # Vanilla, React, Vue examples
└── core/               # npm package "trendcraft"
    ├── src/
    │   ├── core/           # Data normalization, MTF context
    │   ├── indicators/     # Technical indicators (130+)
    │   │   ├── moving-average/  # SMA, EMA, WMA, VWMA, KAMA, T3, HMA, McGinley Dynamic, EMA Ribbon
    │   │   ├── momentum/        # RSI, MACD, Stochastics, DMI/ADX, CCI, ROC, Connors RSI, IMI, ADXR
    │   │   ├── trend/           # Ichimoku, Supertrend, Parabolic SAR
    │   │   ├── volatility/      # Bollinger Bands, ATR, Keltner, Donchian, Choppiness Index
    │   │   ├── volume/          # OBV, MFI, VWAP (Bands), CMF, Volume Profile, Anchored VWAP, Elder Force Index, EMV, Klinger, TWAP, Weis Wave, Market Profile, CVD
    │   │   ├── price/           # Swing Points, Pivot, FVG, BOS, CHoCH, ORB, Gap Analysis, S/R Zone Clustering
    │   │   ├── session/         # ICT Kill Zones, Session Analytics, Session Breakout
    │   │   ├── regime/          # HMM Regime Detection (Baum-Welch, Viterbi)
    │   │   ├── wyckoff/         # VSA (Volume Spread Analysis), Wyckoff Phase Detection
    │   │   ├── relative-strength/
    │   │   └── smc/             # Order Block, Liquidity Sweep
    │   ├── signals/        # Signal detection (crosses, divergence, patterns)
    │   ├── backtest/       # Backtest engine
    │   ├── optimization/   # Grid Search, Walk-Forward, Monte Carlo
    │   ├── scoring/        # Signal scoring system
    │   ├── screening/      # Stock screening
    │   ├── position-sizing/# Position sizing (Kelly, ATR-based, etc.)
    │   ├── meta-strategy/  # Equity Curve Trading, Strategy Rotation
    │   ├── strategy/       # Strategy definition, JSON serialization, condition registry
    │   ├── risk/           # VaR, CVaR, Risk Parity, Correlation-Adjusted Sizing
    │   └── types/          # Type definitions
    ├── bin/             # CLI tools
    ├── docs/            # API docs, guide, cookbook
    ├── cross-validation/ # TA-Lib cross-validation tests
    └── examples/
        ├── chart-viewer/        # Simple chart visualization tool
        ├── trading-simulator/   # React-based trading simulator with backtesting
        ├── alpaca-demo/         # Multi-agent paper trading system (Alpaca API)
        └── candle-former-demo/  # CandleFormer demo
```

## Development Commands

```bash
pnpm install --frozen-lockfile   # Install dependencies (workspace)
pnpm test                        # Run tests (all packages)
pnpm build                       # Build (all packages)
pnpm lint                        # Biome lint
pnpm format                      # Biome format
```

**Install note**: Prefer `pnpm install --frozen-lockfile` over a plain `pnpm install`, especially in a fresh `git worktree`. A plain install has in practice produced an incomplete dependency graph in a new worktree — vue/react types failed to resolve inside vite-plugin-dts (TS2305 errors during `pnpm build`) — even though the lockfile was reported as "up to date". `--frozen-lockfile` refuses to modify the lockfile and in that situation produced a clean, consistent install.

### Package-level commands

```bash
cd packages/core
pnpm test         # Run core tests (vitest)
pnpm build        # Build core (vite)
```

```bash
cd packages/chart
pnpm test         # Run chart tests (vitest, 19 test files)
pnpm build        # Build chart (vite)
pnpm dev          # Dev server with examples
pnpm size-check   # Check bundle size limits (esbuild, brotli-compressed)
```

## General Rules

- Always use English for all code comments, labels, UI text, README files, and documentation unless explicitly told otherwise
- Before making any code changes, confirm understanding of the user's intent. Do not modify code to 'fix' something unless explicitly asked. When the user asks a question about current state, answer the question first — do not preemptively change code

## Code Quality

- If a single file exceeds 500 lines, suggest splitting it into focused modules before adding more code
- All indicators return `Series<T>` type (`{ time: number, value: T }[]`)
- Uses Wilder's smoothing method (RSI, ATR, etc.)
- Functions should have JSDoc comments with @example

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sawapi/trendcraft](https://github.com/sawapi/trendcraft) — distributed by [TomeVault](https://tomevault.io/claim/sawapi).
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
