---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About IARA.jl

IARA (Interaction Assessment between Regulators and Agents) is a comprehensive Julia-based computational model for simulating economic dispatch and price formation in electricity markets. It provides hourly simulations of large-scale systems with detailed representation of generation, transmission, storage, and consumption, enabling assessment of price formation mechanisms based on "cost," "bid," or hybrid models.

**Project:** PSR/CCEE (Brazilian Electricity Trading Chamber) with World Bank funding
**License:** Mozilla Public License 2.0

## Development Commands

### Package Management
```bash
# Activate the IARA.jl environment
julia --project=.

# Install/update dependencies
julia --project=. -e 'using Pkg; Pkg.instantiate()'
```

### Testing
```bash
# Run all tests
julia --project=. test/runtests.jl

# Update test results (when expected outputs change)
julia --project=. test/runtests.jl update_test_results

# Run big (computationally expensive) tests
julia --project=. test/runtests.jl run_big_tests

# Run specific test case
julia --project=. test/case_01/base_case/test_case.jl
```

### Code Formatting
```bash
# Format code using JuliaFormatter
julia --project=. -e 'using JuliaFormatter; format(".")'
```

Configuration in `.JuliaFormatter.toml`:
- Indent: 4 spaces
- Margin: 120 characters
- Always use `return` keyword
- Trailing commas enabled
- Semicolon-separated kwargs

### Running IARA
```bash
# Typical execution with a case directory
julia --project=. -e 'using IARA; IARA.main(["path/to/case"])'

# With Docker
docker pull ghcr.io/psrenergy/iara:latest
```

## Architecture Overview

### Core Module Structure ([src/IARA.jl](src/IARA.jl))

The module loads in a specific order representing dependency layers:

1. **Foundations**: Enumerations (`enumx.jl`), utilities, path handling
2. **Collections**: 16 data model types (see below)
3. **I/O**: Input loading, output writing, time series views
4. **Mathematical Model**: Variables, constraints, objective functions
5. **Algorithms**: SDDP, Nash equilibrium, market clearing, bidding strategies
6. **Post-processing**: Revenue, profit calculations, visualization

### Collections System ([src/collections/](src/collections/))

All entity types inherit from `AbstractCollection` and follow a common pattern:

**Generation Assets:**
- `HydroUnit` - Hydroelectric with cascading, volume tracking, minimum outflow
- `ThermalUnit` - Thermal generation with unit commitment, ramp constraints
- `RenewableUnit` - Wind/solar with curtailment, O&M costs
- `BatteryUnit` - Energy storage with charge/discharge cycles

**Network:**
- `Bus` - Electrical buses with voltage levels
- `Branch` - AC transmission lines with flow limits
- `DCLine` - DC interconnections
- `Interconnection` - Inter-regional connections

**Market Entities:**
- `BiddingGroup` - Strategic bidding entities (market participants)
- `AssetOwner` - Asset ownership for revenue/profit tracking
- `VirtualReservoir` - Aggregated hydro reservoirs for market modeling
- `DemandUnit` - Load centers with elastic/flexible demand support

**Other:**
- `Configurations` - Study parameters and run settings
- `GaugingStation` - Inflow measurement points
- `Zone` - Zonal aggregations

### Run Modes ([src/enumx.jl](src/enumx.jl))

The system supports multiple operational modes via `RunMode` enum:

- **`TRAIN_MIN_COST`** - Train SDDP model, save cuts for later use
- **`MIN_COST`** - Load saved cuts, run fast simulation
- **`MARKET_CLEARING`** - Full market clearing with bidding
- **`SINGLE_PERIOD_MARKET_CLEARING`** - Single period clearing (debugging)
- **`SINGLE_PERIOD_HEURISTIC_BID`** - Generate heuristic bids for one period

### Mathematical Model ([src/mathematical_model.jl](src/mathematical_model.jl))

Uses **action-based dispatch pattern**:
- `AbstractAction` - Base type for all model operations
- `SubproblemAction` - Actions per SDDP node (build/update)
- `ProblemAction` - Actions once per problem (output init/write)

**Model Action Types:**
1. `train_min_cost_model_action` - System cost minimization
2. `price_taker_bid_model_action` - Price-taking bidding agents
3. `price_maker_bid_model_action` - Strategic bidding (convex hull)
4. `market_clearing_model_action` - Full market clearing (hybrid/cost/bid-based)
5. `reference_curve_model_action` - Hydro reference curve generation

Variables and constraints are defined in:
- [src/model_variables/](src/model_variables/) - ~27 variable types
- [src/model_constraints/](src/model_constraints/) - ~31 constraint types

### External Time Series ([src/external_time_series/](src/external_time_series/))

Flexible time series reading system:
- `TimeSeriesView` - Generic time series with period/scenario/subperiod mapping
- `BidsView` - Strategic bid curves and quantities
- `ExAntePostViews` - Handles ex-ante (forecast) vs ex-post (actual) data
- `HourSubperiodMapping` - Aggregation between hourly and subperiod resolution
- Caching for flexible demand

### Data Flow

```
Case Files → load_study() → Database (SQLite via PSRClassesInterface)
    ↓
initialize_collections() → Load time series
    ↓

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psrenergy/IARA.jl](https://github.com/psrenergy/IARA.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
