---
trigger: always_on
description: *This file serves as context for AI assistants. It contains only the most critical information needed to understand and work with the codebase. For detailed documentation, see the `docs/` directory.*
---

# Binance Portfolio Monitor

*This file serves as context for AI assistants. It contains only the most critical information needed to understand and work with the codebase. For detailed documentation, see the `docs/` directory.*

## Core Purpose
Tracks cryptocurrency trading performance by comparing actual portfolio NAV (Net Asset Value) against a passive 50/50 BTC/ETH benchmark strategy.

## How It Works

### 1. Portfolio Tracking
- **Fetches NAV** from Binance futures accounts every hour
- **NAV = spot + futures + funding + earn positions** (all wallets)
- **Tracks deposits/withdrawals** to adjust benchmark accordingly
- Stores everything in Supabase for historical analysis

### 2. Benchmark Logic
The benchmark simulates "what if I just held 50% BTC and 50% ETH":
- **Starts** with same value as initial portfolio NAV
- **Rebalances weekly** (Mondays) to maintain 50/50 allocation
- **Adjusts for cash flows**: When money deposited/withdrawn, benchmark buys/sells proportionally
- **Formula**: `benchmark_value = btc_units × current_btc_price + eth_units × current_eth_price`

### 3. Performance Calculation
```
Trading Alpha = (Current NAV / Benchmark Value - 1) × 100%
```
- **Positive %**: Your trading beats passive holding
- **Negative %**: You'd be better off just holding BTC/ETH

## Critical Implementation Details

### Transaction Processing
- **Problem**: Avoid double-counting historical deposits when restarting
- **Solution**: `initialized_at` timestamp - only process transactions after initialization
- **Unique constraint**: (account_id, transaction_id) prevents duplicates

### Deposit Processing & Multi-Coin Support
- **Supports all cryptocurrencies** - not limited to BTC/ETH
- **Automatic USD conversion** with fallback mechanism:
  1. Try direct USDT pair (e.g., SOLUSDT)
  2. Fall back to BTC routing (SOL→BTC→USD)
  3. Mark as `price_missing` if no price available
- **Metadata storage** for each deposit:
  ```json
  {
    "coin": "SOL",
    "network": "SOL",
    "usd_value": 500.00,
    "coin_price": 100.00,
    "price_source": "direct",  // or "via_btc"
    "price_missing": false
  }
  ```
- **Cashflow handling**: Deposits with missing prices are stored but excluded from NAV calculations
- **Retroactive updates**: Utility scripts can update historical deposits with missing prices

### Geographic Restrictions
- **Binance blocks cloud IPs** (AWS, Vercel, etc.)
- **Solution**: Use `data-api.binance.vision` for public data (prices)
- **Private data** (account NAV) works through Frankfurt region

### Process Safety
- **Lock file**: `/tmp/.binance_monitor.lock` prevents duplicate runs
- **Auto-recovery**: Stale locks (>1 hour) are cleared automatically

### Alpha Calculation & Fee Management
- **TWR (Time-Weighted Returns)**: Eliminates deposit/withdrawal bias
- **Alpha = Portfolio TWR - Benchmark TWR**
- **HWM (High Water Mark)**: Adjusted for all cashflows
- **Performance Fee**: Configurable per account (default 50%)
  - Only charged when NAV > HWM AND alpha > 0
  - No management fees
  - Monthly accruals, separate collection tracking

### Fee Configuration
```json
"fee_management": {
  "default_performance_fee_rate": 0.50,
  "calculation_schedule": "monthly",  // or "daily", "hourly"
  "calculation_day": 1,
  "calculation_hour": 0,
  "test_mode": { "enabled": false }
}
```
- Each account has `performance_fee_rate` in database
- Manual calculation: `python scripts/run_fee_calculation.py`

## Key Files
- `api/index.py` - Core monitoring logic
- `api/dashboard.py` - Web UI (port 8000) 
- `api/fee_calculator.py` - Fee calculations with flexible scheduling
- `api/calculate_fees.py` - Cron endpoint for fee calculations
- `scripts/run_fee_calculation.py` - Manual fee calculation tool
- `config/settings.json` - Configuration including fee management
- `deployment/aws/run_forever.py` - Production runner

## Database Schema
```
binance_accounts → API credentials + performance_fee_rate per account
benchmark_configs → tracks BTC/ETH units per account with rebalancing history + audit columns
nav_history → hourly NAV and benchmark values with account names
processed_transactions → deposit/withdrawal history (uses 'type' field)
fee_tracking → fee accruals and collections
system_logs → operation logs with retention
price_history → BTC/ETH price snapshots
account_processing_status → tracks last processed timestamp for each account
system_metadata → system-wide configuration and metadata
users → user authentication data
benchmark_rebalance_history → complete history of all rebalancing operations with calculations
benchmark_modifications → tracks all deposit/withdrawal impacts on benchmark

Views:
nav_with_cashflows → NAV data enriched with transactions
period_returns → TWR period returns calculation
hwm_history → High Water Mark tracking

Functions:
calculate_twr_period() → TWR for any date range
calculate_monthly_fees() → fee calculation with account-specific rates
```

### Transaction Processing
- **Field**: Use `type` field (NOT `transaction_type`) in processed_transactions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frla18cz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
