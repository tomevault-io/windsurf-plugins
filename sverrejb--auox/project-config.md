---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Auox (Aurum Oxydatum) is a terminal-based banking application written in Rust that interfaces with SpareBank 1's API. It provides a TUI (Terminal User Interface) for viewing and managing bank accounts using OAuth authentication.

## Commands

### Build and Run
```bash
cargo build          # Build the project
cargo run            # Run the application
cargo check          # Quick type checking without building
cargo clippy         # Run linter
```

### Testing
```bash
cargo test           # Run all tests
cargo test <name>    # Run a specific test
```

## Architecture

### Module Structure
- `src/main.rs`: Application entry point and TUI event loop
- `src/auth.rs`: OAuth authentication flow implementation
- `src/fileio.rs`: Configuration and token file management
- `src/api.rs`: SpareBank 1 API client functions
- `src/transfers.rs`: Transfer business logic
- `src/models/`: Data models for API responses
  - `accounts.rs`: Account data structures
  - `token.rs`: Token data structures
  - `transactions.rs`: Transaction data structures
  - `transfers.rs`: Transfer DTOs and response structures
- `src/ui.rs`: TUI rendering logic

### OAuth Authentication Flow
The application implements a three-tiered OAuth authentication strategy in `src/auth.rs`:

1. **Access Token Check**: First attempts to use stored access token from `auth.json`
2. **Refresh Token Flow**: If access token invalid, attempts to refresh using stored refresh token
   - Makes POST request to `/oauth/token` with `grant_type=refresh_token`
   - Saves new token data to `auth.json` on success
3. **Full OAuth Flow**: If refresh fails, initiates full OAuth flow with SpareBank 1's API:
   - Spawns local HTTP server on port 8321 to receive OAuth callback
   - Opens browser to SpareBank 1's authorization endpoint with `finInst` parameter from config
   - Waits for authorization code via redirect
   - Exchanges code for access token

### File Management (`src/fileio.rs`)
- **Config file location**:
  - macOS: `~/Library/Application Support/auox/config.toml`
  - Linux: `~/.config/auox/config.toml`
  - Windows: `%APPDATA%\auox\config.toml`
- **Token file location**:
  - macOS: `~/Library/Application Support/auox/auth.json`
  - Linux: `~/.local/share/auox/auth.json`
  - Windows: `%APPDATA%\auox\auth.json`
- Creates directories and config template automatically on first run if they don't exist
- **Required config fields**:
  - `client_id`: OAuth client ID for SpareBank 1 API
  - `client_secret`: OAuth client secret for SpareBank 1 API
  - `financial_institution`: Financial institution ID (e.g., `fid-smn` for SpareBank 1 Midt-Norge)
- **Token file structure** (`auth.json`):
  - `access_token`: Current access token
  - `expires_in`: Access token expiry time (seconds)
  - `refresh_token`: Token for refreshing access
  - `refresh_token_expires_in`: Refresh token expiry time
  - `refresh_token_absolute_expires_in`: Absolute refresh token expiry
  - `token_type`: Token type (Bearer)

### TUI Architecture (`src/main.rs`, `src/ui.rs`)
- Built with `ratatui` and `crossterm` for terminal UI
- Main loop pattern: enable raw mode → render loop → cleanup
- **Multiple views**:
  - `Accounts`: Main account list view
  - `Menu`: Action menu for selected account
  - `Transactions`: Transaction history for selected account
  - `TransferSelect`: Select destination account for transfer
  - `TransferModal`: Enter transfer amount and optional message
- **Event handling**:
  - `Ctrl+C`: Exit application with dissolve animation effect
  - `Esc`: Navigate back in view stack
  - `Enter`: Select item / confirm action
  - `Up/Down arrows`: Navigate lists with modulo wrap-around
  - `b`: Toggle balance visibility
  - `m`: Toggle credit card visibility
  - `Tab`: Switch between input fields (in transfer modal)
- State management via `TableState` and `ListState` for tracking selections
- UI rendered in `ui::draw()` with blue highlight style for selected items
- Visual effects powered by `tachyonfx` (coalesce in, dissolve out)

### API Client (`src/api.rs`)
- `get_accounts()`: Fetches account list from `/personal/banking/accounts`
- `get_transactions(account_key)`: Fetches transaction history from `/personal/banking/transactions`
- `create_transfer(transfer)`: Creates regular account transfer via `/personal/banking/transfer/debit`
- `create_credit_card_transfer(transfer)`: Creates credit card transfer via `/personal/banking/transfer/creditcard/transferTo`
- `perform_transfer(app)`: High-level transfer logic that automatically detects credit cards and routes to appropriate endpoint
- Uses Bearer token authentication
- Returns structured response types with error handling

### Data Models (`src/models/`)
Defines SpareBank 1 API response structures:
- **accounts.rs**:
  - `AccountData`: Top-level response with accounts array and errors
  - `Account`: Bank account with balance, IBAN, owner info, account properties, and credit card ID
  - `AccountProperties`: Detailed flags for account capabilities (transfers, payments, special account types)
- **transactions.rs**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sverrejb/auox](https://github.com/sverrejb/auox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
