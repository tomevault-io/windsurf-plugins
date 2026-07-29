---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Rust-based CLI (`longbridge`) that wraps every Longbridge OpenAPI endpoint for scripting, AI-agent tool-calling, and daily trading workflows. Also ships a full-screen TUI for interactive market monitoring.

## Core Architecture

### Tech Stack

- **UI Framework**: Ratatui (v0.24.0) - TUI rendering
- **Async Runtime**: Tokio (v1.33.0) - Async I/O
- **ECS Framework**: Bevy ECS (v0.11) - Entity-Component-System architecture
- **Market SDK**: longbridge (v4.0.0) - Longbridge OpenAPI Rust SDK (dependency alias: `longbridge-sdk`)
- **State Management**: DashMap, Atomic, RwLock - Thread-safe global state

### Key Modules

#### 1. `src/auth.rs` - Auth utilities

- `clear_token()` - Clear stored OAuth token (logout). Deletes token file used by the SDK.
- OAuth and token refresh are handled by the longbridge SDK: use `longbridge::oauth::OAuthBuilder` in `openapi::context::init_contexts()`. Token is loaded from `~/.longbridge/openapi/tokens/<client_id>` or browser flow is started; the SDK auto-refreshes the token.
- Local callback server: default port `60355` (configurable via `OAuthBuilder::callback_port()`)

#### 2. `src/openapi/` - OpenAPI Integration Layer

- `context.rs` - Global context management
  - `init_contexts()` - Initialize QuoteContext and TradeContext with OAuth token, returns WebSocket receiver
  - `quote()` - Get global QuoteContext (for quotes, subscriptions)
  - `trade()` - Get global TradeContext (for trading operations)
  - Uses `OnceLock` for global singleton

#### 3. `src/data/` - Data Layer

- `types.rs` - Base type definitions
  - `Counter` - Stock identifier (format: `700.HK`, `AAPL.US`)
  - `TradeStatus`, `Currency`, `Market` - Enum types
  - `QuoteData`, `Candlestick`, `Depth` - Market data structures
- `stock.rs` - Stock data structure
  - `update_from_quote()` - Update from longbridge quote
  - `update_from_depth()` - Update from longbridge depth
- `stocks.rs` - Global stock cache (based on `DashMap`)
  - `STOCKS` - Global singleton, provides `get()`, `mget()`, `insert()`, `modify()` methods

#### 4. `src/app.rs` - Application Main Loop

- Uses Bevy ECS to manage app state (`AppState`)
- Handles UI updates via `mpsc::unbounded_channel`
- Subscribes to index quotes (HSI, DJI, Shanghai Composite, etc.)
- Integrates search, selection, popup components

#### 5. `src/system.rs` - System Logic and UI Rendering

- Contains rendering logic for pages (Watchlist, Stock, Portfolio, etc.)
- Handles user input and state transitions

#### 6. `src/api/` - API Call Layer

- `search.rs` - Stock search
- `quote.rs` - Quote queries
- `account.rs` - Account information
- Uses `openapi::quote()` and `openapi::trade()`

#### 7. `src/widgets/` and `src/views/` - UI Components

- `Terminal` - Terminal management
- `Search`, `LocalSearch` - Search components
- `Carousel` - Carousel component
- `Loading` - Loading animation
- Various popups and navigation components

### Data Flow

1. **Authentication**: `main.rs` → `openapi::init_contexts()` → `longbridge::oauth::OAuthBuilder::build()` (loads token from disk or browser flow) → `Config::from_oauth(oauth)` → SDK handles token refresh automatically
2. **Initialization**: `main.rs` → `openapi::init_contexts()` → QuoteContext and TradeContext created with config → Get WebSocket receiver
3. **Subscribe Quotes**: `app.rs` → `openapi::quote().subscribe()` → longbridge SDK
4. **Receive Push**: WebSocket receiver → Parse `PushEvent` → Update `STOCKS` cache
5. **UI Rendering**: Bevy ECS systems → Read `STOCKS` → Ratatui rendering

## Development Commands

### Build and Run

```bash
# Development build
cargo build

# Release build (with LTO and optimizations)
cargo build --release

# Run
cargo run
```

### Code Checks

```bash
# Clippy check (project uses strict pedantic rules)
cargo clippy

# Format
cargo fmt
```

**Before every `git push` or `gh pr create`, always run both and fix all issues:**

```bash
cargo fmt && cargo clippy
```

### Verifying Changes

After any data-layer or CLI output change, verify correctness by comparing the installed release binary against the local build using the same command and `--format json`:

```bash
# Run both and compare — output should be identical (timestamps may differ for live data)
longbridge <command> <args> --format json
cargo run -- <command> <args> --format json
```

Pick commands that exercise the modified code paths. Common ones:

| Changed area             | Verification command                         |
| ------------------------ | -------------------------------------------- |
| Trade direction / trades | `longbridge trades 700.HK --format json`     |
| Kline / AdjustType       | `longbridge kline 700.HK --format json`      |
| Quote / calc-index       | `longbridge calc-index 700.HK --format json` |
| Static info              | `longbridge static 700.HK --format json`     |

### Configuration

**Authentication Method: OAuth 2.0 (longbridge SDK)**

The application uses the longbridge SDK's built-in OAuth. On first run:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [longbridge/longbridge-terminal](https://github.com/longbridge/longbridge-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
