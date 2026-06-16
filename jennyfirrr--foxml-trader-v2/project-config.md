---
trigger: always_on
description: Tick-level crypto trading engine in C++. Branchless fixed-point arithmetic, bitmap-based portfolio management, regression-driven gate adjustment with regime detection. Per-core sharded hot path (40-400ns p99), single-symbol producer thread fanning real Binance ticks across SPSC rings to per-core consumers.
---

# CLAUDE.md

## Overview

Tick-level crypto trading engine in C++. Branchless fixed-point arithmetic, bitmap-based portfolio management, regression-driven gate adjustment with regime detection. Per-core sharded hot path (40-400ns p99), single-symbol producer thread fanning real Binance ticks across SPSC rings to per-core consumers.

**Sharded is production. Legacy single_core LIVE is deprecated (warned at boot). Legacy backtest is gone — `Backtest_Run` wraps `BacktestSharded_Run`.**

## Build

`./build.sh test` (engine + controller_test), `gui` (engine_gui + foxml_suite), `suite` (suite with XGBoost), `tsan` / `asan` (sanitizer builds), `all`, `clean`. Build flags: `-DLATENCY_PROFILING=ON`, `-DLATENCY_LITE=ON`, `-DLATENCY_BENCH=ON`, `-DBUSY_POLL=ON`, `-DUSE_NATIVE_128=ON`.

Build dirs (different compile flags → different outputs): `build/` (ANSI + tests, zero deps), `build_gui/` (engine_gui + foxml_suite — SDL2 + OpenGL3 + ImGui), `build_suite/` (same + XGBoost), `build_lat/` (LATENCY_PROFILING), `build_tsan/`, `build_asan/`.

XGBoost C library (for `-DUSE_XGBOOST=ON`): clone `dmlc/xgboost` recursive, cmake with `-DBUILD_STATIC_LIB=OFF`, install + ldconfig.

`build.sh` symlinks `engine.cfg` into each build dir; `bin/engine_gui` → `build_gui/engine_gui`.

## Architecture (sharded)

N cores (default 4, cap 16), each = self-contained strategy unit (slow + hot pthread pair). Producer thread fans Binance ticks across per-core SPSC rings; per-core hot-paths consume; per-core slow-paths rebuild gate parameters on cadence.

- Per-core strategy (`core_N_strategy=simple_dip|momentum|ema_cross|ml`)
- Per-core ML model (`core_N_model_path=...` or `core_N_model_dir=...`)
- Per-core risk (`core_N_risk_pct=...`)
- Per-core ConfidenceScorer (when STRATEGY_ML)
- Per-core slow_state owns rolling/regime/flow data (v5.1.2+)
- Hot path p99 ≤500ns
- Partial exits (`partial_exit_enabled=1`): each core owns 2 slots (legs A+B); max cores = 8
- `engine_arch=per_core_slow` (default v5.0+) | `centralized` (legacy)

```
HOT PATH (per tick, per core, branchless):
  BG_Evaluate → SG_Evaluate ×2 → TradeEvent push (rare branch)

SLOW PATH (per-core thread, every poll_interval ticks):
  EventLoop_UpdateRollingStateOneCore → RebuildOneCore (regime + strategy)
  → ExecutionCore_SetParameters (seqlock to hot path)
  → TimeExitOneCore → TrailingSLRatchetOneCore

GLOBAL THREADS:
  Producer: tick read + fan_out + ema_price replication + GUI publish
  Drainer:  OMS_DrainSubmit + OrderManager_Tick + DrainPostFill
  Async:    Binance trade WS, depth WS, Tick/DepthRecorder, Notify worker, GUI
```

## Data Flow: Regime Detection

```
Per-engine slow_state (RollingStats × 4 + RORRegressor + flow + depth) →
  RegimeSignals (slope, R², variance, ror_slope, ema_sma_spread,
                 book_imbalance, spread_z, flow_*_ewma, large_trade_z, ...) →
  Regime_Classify (trending_score + volatile_score with hysteresis) →
    RANGING / TRENDING / VOLATILE / MILD_TREND → strategy dispatch
```

`RegimeSignals` is the extensibility point — see `DOCS/CLAUDE_INTEGRATION.md` for the recipe.

## Directory Structure

- **CoreFrameworks/** — OrderGates, Portfolio, ExecutionCore, ControllerEventLoop, EngineSharded, ShardedSnapshot/Persist, GateParameters, TradeEvent, OrderManager, ShardedBacktestDriver
- **Strategies/** — RegimeDetector, MeanReversion, Momentum, SimpleDip, EmaCross, MLStrategy, StrategyParameters (dispatcher), StrategyInterface
- **DataStream/** — BinanceCrypto/Depth, DepthReplayState, DepthRecorder, TickRecorder, BinanceOrderAPI, EngineTUI
- **FixedPoint/** — FPN<F=64> (4096-bit)
- **MemHeaders/** — PoolAllocator (bitmap order pool), BuddyAllocator
- **ML_Headers/** — RollingStats, ROR_regressor, ConfidenceScore, ModelInference (XGBoost), FlowFeatures
- **GUI/** — Dear ImGui native: FoxmlTheme, DashboardPanels, ChartPanel, CandleAccumulator, TradeReader, SettingsPanel, TradeHistoryPanel, LogViewerPanel, GuiThread
- **Backtest/** — `Backtest_Run` wrapper + `BacktestSharded_Run`, BacktestPanels, LabelFunctions, HeldOutSplit, ValidationSplit
- **tests/** — controller_test.cpp (739+ assertions), parity_harness.cpp
- **DOCS/** — CHANGELOG.md, changelogs/, CLAUDE_*.md (split-load reference docs)
- **plans/** — gitignored, working plans
- **Version.hpp**, **Limits.hpp** — single source of truth

## Code Conventions

- `using namespace std;` throughout
- C-style with templates, no classes (with one explicit exception:
  RAII destructors on resource-owning structs that own threads or
  mmap'd memory, e.g. `~OrderManagerState()` since v5.11.26 — see
  the destructor comment in `CoreFrameworks/OrderManager.hpp` for
  the criteria; default is still no destructor)
- `Pattern_FunctionName` (e.g. `Portfolio_Init`, `BG_Evaluate`)
- Hot-path math is `FPN<F>` only, no floats (F=64 = 4096-bit)
- Branchless: mask tricks `-(uint64_t)pass`, word-level mask-select
- Inline comments explain reasoning, not what
- **Preserve user's voice in existing comments when editing**

### Test file size discipline (added v5.11.35)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jennyfirrr/FoxML_Trader_v2](https://github.com/Jennyfirrr/FoxML_Trader_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
