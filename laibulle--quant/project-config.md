---
trigger: always_on
description: Create `quant` - A comprehensive Elixir library for fetching financial and cryptocurrency data from multiple providers and loading it directly into Explorer DataFrames for high-performance analysis.
---

# AGENT.md - quant Project Instructions

## Project Overview

Create `quant` - A comprehensive Elixir library for fetching financial and cryptocurrency data from multiple providers and loading it directly into Explorer DataFrames for high-performance analysis.

## Core Vision

- **Multi-Provider Architecture**: Support Yahoo Finance, Alpha Vantage, Binance, CoinGecko, and other providers
- **Explorer-First**: All data returns as Explorer DataFrames for immediate analysis
- **Performance-Focused**: Leverage Explorer's Polars backend for speed
- **Developer-Friendly**: Simple, consistent API across all providers
- **Extensible**: Easy to add new data providers
- **Few dependencies**: As few dependencies to external libraries as possible

## Project Structure

```
quant/
├── lib/
│   ├── quant.ex                    # Main API module
│   ├── quant/
│   │   ├── providers/                     # Data provider implementations
│   │   │   ├── behaviour.ex               # Provider behaviour definition
│   │   │   ├── yahoo_finance.ex           # Yahoo Finance implementation
│   │   │   ├── alpha_vantage.ex           # Alpha Vantage implementation
│   │   │   ├── binance.ex                 # Binance crypto data
│   │   │   ├── coin_gecko.ex              # CoinGecko crypto data
│   │   │   └── twelve_data.ex             # Twelve Data implementation
│   │   ├── rate_limiting/                 # Advanced rate limiting system
│   │   │   ├── behaviour.ex               # Rate limiter behaviour definition
│   │   │   ├── ets_backend.ex             # ETS-based rate limiter
│   │   │   ├── redis_backend.ex           # Redis-based rate limiter
│   │   │   └── provider_config.ex         # Provider-specific configurations
│   │   ├── http_client.ex                 # HTTP client wrapper
│   │   ├── rate_limiter.ex                # Advanced rate limiter manager
│   │   ├── cache.ex                       # Caching layer (ETS-based)
│   │   ├── data_transformer.ex            # Data normalization utilities
│   │   └── config.ex                      # Configuration management
│   └── quant/
│       └── application.ex                 # OTP application
├── test/
│   ├── fin_explorer_test.exs
│   ├── providers/
│   │   ├── yahoo_finance_test.exs
│   │   ├── alpha_vantage_test.exs
│   │   ├── binance_test.exs
│   │   └── coin_gecko_test.exs
│   └── support/
│       ├── fixtures/                      # Mock API responses
│       └── test_helper.ex
├── config/
│   ├── config.exs                         # Application configuration
│   ├── dev.exs                           # Development config
│   ├── test.exs                          # Test config
│   └── runtime.exs                       # Runtime config
├── docs/                                  # Documentation
│   ├── providers/                         # Provider-specific docs
│   └── examples/                          # Usage examples
├── mix.exs                               # Project definition
├── README.md                             # Project README
├── CHANGELOG.md                          # Version history
└── LICENSE                               # MIT License
```

## Main API Design

### Core Interface

```elixir
# Simple single symbol fetch
{:ok, df} = Quant.Explorer.fetch("AAPL", provider: :yahoo_finance)

# Multiple symbols
{:ok, df} = Quant.Explorer.fetch(["AAPL", "MSFT", "GOOGL"], provider: :alpha_vantage)

# Crypto data
{:ok, df} = Quant.Explorer.fetch("BTC-USD", provider: :binance)

# With options
{:ok, df} = Quant.Explorer.fetch("AAPL", 
  provider: :yahoo_finance,
  period: "1y",
  interval: "1d",
  start_date: ~D[2023-01-01],
  end_date: ~D[2024-01-01]
)

# Real-time quotes
{:ok, df} = Quant.Explorer.quote("AAPL", provider: :yahoo_finance)

# Company information
{:ok, info} = Quant.Explorer.info("AAPL", provider: :yahoo_finance)

# Search functionality
{:ok, df} = Quant.Explorer.search("Apple", provider: :yahoo_finance)
```

### Provider-Specific Functions

```elixir
# Provider modules can be called directly for advanced usage
{:ok, df} = Quant.Explorer.Providers.YahooFinance.history("AAPL", period: "1y")
{:ok, df} = Quant.Explorer.Providers.Binance.klines("BTCUSDT", interval: "1h")
```

## Key Requirements

### 1. Dependencies (mix.exs)

```elixir
defp deps do
  [
    {:explorer, "~> 0.11"},
    {:decimal, "~> 2.0"},
    {:telemetry, "~> 1.0"},
    {:ex_doc, "~> 0.31", only: :dev, runtime: false},
    {:credo, "~> 1.7", only: [:dev, :test], runtime: false},
    {:dialyxir, "~> 1.3", only: [:dev], runtime: false},
    {:bypass, "~> 2.1", only: :test}
  ]
end
```

### 2. Provider Behaviour

```elixir
defmodule Quant.Explorer.Providers.Behaviour do
  @moduledoc """
  Behaviour that all data providers must implement.
  """
  
  alias Explorer.DataFrame
  
  @type symbol :: String.t()
  @type symbols :: [symbol()]
  @type options :: keyword()
  @type period :: String.t()
  @type interval :: String.t()
  
  @callback history(symbol() | symbols(), options()) :: 
    {:ok, DataFrame.t()} | {:error, term()}
    
  @callback quote(symbol() | symbols()) :: 
    {:ok, DataFrame.t()} | {:error, term()}
    
  @callback info(symbol()) :: 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laibulle/quant](https://github.com/laibulle/quant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
