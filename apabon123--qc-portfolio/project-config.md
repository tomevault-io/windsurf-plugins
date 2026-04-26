---
trigger: always_on
description: **ALWAYS CHECK QUANTCONNECT'S BUILT-IN CAPABILITIES BEFORE IMPLEMENTING CUSTOM SOLUTIONS**
---

# QuantConnect Three-Layer CTA Framework - Cursor Rules

## 🎯 **CORE PRINCIPLES**

### **0. LEAN-FIRST DEVELOPMENT** 🚨 **PRIMARY DIRECTIVE**
**ALWAYS CHECK QUANTCONNECT'S BUILT-IN CAPABILITIES BEFORE IMPLEMENTING CUSTOM SOLUTIONS**

### **⚠️ CRITICAL: QUANTCONNECT SYMBOL OBJECT LIMITATION** 🚨 **NEVER IGNORE**
**NEVER pass QuantConnect Symbol objects as constructor parameters - causes "error return without exception set"**

#### **The Problem**
QuantConnect Symbol objects get wrapped in `clr.MetaClass` which cannot be passed to Python constructors.
This causes the infamous "error return without exception set" error that crashes algorithm initialization.

#### **Forbidden Patterns** ❌
```python
# ❌ NEVER DO THIS - Will cause constructor crashes
def __init__(self, symbol_objects):
    self.symbols = symbol_objects  # Symbol objects cause clr.MetaClass issues

# ❌ NEVER DO THIS - Passing Symbol objects between components  
component = MyComponent(self, config_manager, shared_symbols)  # shared_symbols contains Symbol objects

# ❌ NEVER DO THIS - Storing Symbol objects in shared data structures
shared_data = {'symbols': [symbol1, symbol2, symbol3]}  # Symbol objects will crash constructors
```

#### **Correct Patterns** ✅
```python
# ✅ Use string identifiers instead
def __init__(self, symbol_strings):
    self.symbol_strings = symbol_strings  # ['ES', 'NQ', 'ZN'] - safe to pass

# ✅ Convert Symbol objects to strings before passing
symbol_strings = [str(symbol) for symbol in symbol_objects]
component = MyComponent(self, config_manager, symbol_strings)

# ✅ Use QC's native methods directly - let QC handle Symbol creation
future = self.AddFuture("ES", Resolution.Daily)  # QC creates Symbol internally
self.futures_symbols.append(future.Symbol)       # Store QC-created Symbol
```

#### **Solution: Use QC Native Universe Management**
```python
# ✅ CORRECT - Direct QC native approach (no custom Symbol management)
def _setup_futures_universe(self):
    for symbol_str in ['ES', 'NQ', 'ZN']:
        future = self.AddFuture(symbol_str, Resolution.Daily)  # QC handles Symbol creation
        self.futures_symbols.append(future.Symbol)            # Store QC-created Symbol
```

**Reference**: [QuantConnect Forum - Python Custom Exception Issue](mdc:https:/www.quantconnect.com/forum/discussion/9331/python-custom-exception-definition-does-not-seem-to-work)

#### **MANDATORY LEAN CAPABILITY CHECK**
Before writing ANY trading, financial, or data processing code:
1. **CHECK**: Does QC/LEAN already provide this functionality?
2. **VERIFY**: Review the LEAN API documentation
3. **CONFIRM**: Search existing QC methods in AlgorithmImports
4. **IMPLEMENT**: Use LEAN's native methods, not custom implementations

#### **COMMON LEAN CAPABILITIES (Use These, Don't Reinvent)**
```python
# Portfolio Management - NEVER build custom portfolio tracking
self.Portfolio.TotalPortfolioValue          # Total portfolio value
self.Portfolio[symbol].Quantity             # Position quantity
self.Portfolio[symbol].HoldingsValue        # Position value
self.Portfolio[symbol].UnrealizedProfit     # Unrealized P&L
self.Portfolio.Cash                         # Available cash
self.Portfolio.TotalMarginUsed              # Margin usage

# Technical Indicators - NEVER build custom indicators
self.SMA(symbol, period)                    # Simple Moving Average
self.EMA(symbol, period)                    # Exponential Moving Average  
self.RSI(symbol, period)                    # Relative Strength Index
self.MACD(symbol, fast, slow, signal)       # MACD
self.BB(symbol, period, std)                # Bollinger Bands
self.ATR(symbol, period)                    # Average True Range
self.AROON(symbol, period)                  # Aroon Oscillator
self.CCI(symbol, period)                    # Commodity Channel Index

# Order Management - NEVER build custom order systems
self.MarketOrder(symbol, quantity)          # Market orders
self.LimitOrder(symbol, quantity, price)    # Limit orders
self.StopMarketOrder(symbol, quantity, stop)# Stop orders
self.SetHoldings(symbol, percentage)        # Position sizing
self.Liquidate(symbol)                      # Close positions
self.CalculateOrderQuantity(symbol, target) # Calculate quantities

# Data Access - NEVER build custom data fetching
self.History(symbol, bars, resolution)      # Historical data
self.Securities[symbol].Price               # Current price
self.Securities[symbol].HasData             # Data availability
self.Securities[symbol].IsTradable          # Trading status
self.Securities[symbol].Mapped              # Mapped contract (futures)

# Scheduling - NEVER build custom timers/schedulers
self.Schedule.On(date_rule, time_rule, action)  # Custom scheduling
self.DateRules.EveryDay(symbol)                 # Daily scheduling
self.DateRules.Every(DayOfWeek.Monday)          # Weekly scheduling
self.TimeRules.AfterMarketOpen(symbol, minutes) # Market open timing
self.TimeRules.BeforeMarketClose(symbol, minutes) # Market close timing

# Universe Selection - NEVER build custom universe logic
self.AddEquity(symbol, resolution)          # Add equity
self.AddForex(symbol, resolution)           # Add forex
self.AddFuture(symbol, resolution)          # Add futures (continuous)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/apabon123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
