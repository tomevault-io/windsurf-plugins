---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A modern, high-performance Zig library for fetching and parsing OHLCV (Open-High-Low-Close-Volume) financial data from remote CSV files. The library provides:

- **33 technical indicators** covering trend, momentum, volatility, volume, and advanced trading systems
- **Multiple data sources** (HTTP, local files, in-memory)
- **High-performance parsing** with streaming support for large datasets
- **Advanced memory management** with pooling and arena allocation
- **Comprehensive benchmarking** and performance monitoring
- **Clean, explicit API** with Zig's memory safety guarantees

## Build and Development Commands

### Core Commands
```bash
# Build the library and demo
zig build

# Run the demo application
zig build run

# Run all tests
zig build test

# Run benchmarks
zig build benchmark                # Basic benchmark
zig build benchmark-performance     # Comprehensive performance tests
zig build benchmark-streaming       # Streaming vs non-streaming comparison

# Run memory profiler
zig build profile-memory
```

### Testing Individual Components
```bash
# Run specific test file (example)
zig test test/unit/test_time_series.zig -I lib --dep ohlcv -Mohlcv=lib/ohlcv.zig
```

## Architecture Overview

### Module Organization
The library is exposed through `lib/ohlcv.zig` which re-exports all public APIs. The codebase follows a clean separation of concerns:

1. **Data Sources** (`lib/data_source/`)
   - `DataSource` interface for polymorphic data access
   - `HttpDataSource` for fetching from URLs
   - `FileDataSource` for local files  
   - `MemoryDataSource` for in-memory data

2. **Parsing** (`lib/parser/`)
   - `CsvParser` handles CSV parsing with robust error handling
   - `StreamingCsvParser` processes large files in chunks without full memory load
   - `fast_parser.zig` provides optimized parsing primitives with SIMD-aware line counting
   - Skips invalid rows, headers, pre-1970 dates, and zero values
   - Supports multiple line endings (CRLF, LF, CR)

3. **Time Series** (`lib/utils/time_series.zig`)
   - Core container for OHLCV data
   - Provides slicing, filtering, sorting, and transformation operations
   - Memory-safe with explicit allocation/deallocation

4. **Memory Management** (`lib/utils/`)
   - `MemoryPool` provides reusable memory allocation pools
   - `IndicatorArena` offers arena allocation for batch calculations
   - Reduces allocation overhead in hot paths
   - Improves cache locality for performance-critical operations

5. **Technical Indicators** (`lib/indicators/`)
   - **33 indicators** including:
     - **Trend**: SMA, EMA, WMA, ADX, DMI, Parabolic SAR
     - **Momentum**: RSI, MACD, Stochastic, Williams %R, ROC, TRIX, Ultimate Oscillator
     - **Volatility**: ATR, Bollinger Bands, Keltner Channels, Donchian Channels, Price Channels
     - **Volume**: OBV, MFI, CMF, Force Index, Accumulation/Distribution, VWAP, CCI
     - **Advanced**: Ichimoku Cloud, Heikin Ashi, Pivot Points, Elder Ray, Aroon, Zig Zag
   - Each indicator implements a `calculate()` method returning `IndicatorResult` or specialized result types
   - Multi-line indicators (MACD, Bollinger Bands, etc.) return structured results with multiple arrays

### Memory Management Pattern
All allocations are explicit using Zig's allocator pattern:
- Functions accepting allocators return owned memory
- Caller is responsible for calling `.deinit()` on returned structures
- TimeSeries and IndicatorResult have `.deinit()` methods

### Error Handling
- `ParseError` enum for parsing failures
- `FetchError` for data retrieval issues
- Functions return error unions (`!Type`) for explicit error handling

## Key Design Patterns

### Data Flow
```
DataSource -> fetch() -> raw bytes -> CsvParser/StreamingCsvParser -> OhlcvRow[] -> TimeSeries -> Indicators -> IndicatorResult
                                              |
                                              └─> MemoryPool/IndicatorArena (optional optimization)
```

### Preset Data Sources
The library provides preset configurations for common datasets:
- `.btc_usd` - Bitcoin/USD data
- `.sp500` - S&P 500 index
- `.eth_usd` - Ethereum/USD data
- `.gold_usd` - Gold/USD data

These can be fetched either from GitHub or local CSV files in the `data/` directory.

### Testing Strategy
- **Unit tests** in `test/unit/` for individual components:
  - `test_time_series.zig` - TimeSeries container functionality
  - `test_data_sources.zig` - Data source implementations
  - `test_csv_parser.zig` - CSV parsing and streaming
  - `test_indicators.zig` - All 33 technical indicators
  - `test_edge_cases.zig` - Error conditions and edge cases
- **Integration tests** in `test/integration/` for end-to-end workflows
- **Test helpers** in `test/test_helpers.zig` for common utilities
- **Performance tests** via benchmark suite with memory leak detection
- All tests imported through `test/test_all.zig` for unified execution

## Code Conventions

### Naming
- Files: snake_case (e.g., `csv_parser.zig`)
- Types: PascalCase (e.g., `OhlcvRow`, `TimeSeries`)
- Functions: camelCase (e.g., `sliceByTime`, `calculate`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mario-SO/ohlcv](https://github.com/Mario-SO/ohlcv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
