---
trigger: always_on
description: This is a Cargo workspace containing 5 crates under `crates/`, providing a command-line HTTP client with built-in [MPP](https://mpp.dev) payment support, wallet identity management, wallet-backed cards, and a release signing tool. The top-level `tempo` launcher lives in the main tempo repo (`tempo/crates/ext/`).
---

# AGENTS.md

## Repository Overview

This is a Cargo workspace containing 5 crates under `crates/`, providing a command-line HTTP client with built-in [MPP](https://mpp.dev) payment support, wallet identity management, wallet-backed cards, and a release signing tool. The top-level `tempo` launcher lives in the main tempo repo (`tempo/crates/ext/`).

**Supported Payment Protocols:**
- [Machine Payments Protocol (MPP)](https://mpp.dev) - Open protocol for HTTP-native machine-to-machine payments

### Workspace Structure

The root `Cargo.toml` is workspace-only (no package). All dependencies are declared as `[workspace.dependencies]` in the root and consumed via `dep.workspace = true` in each crate. All crates live under `crates/`:

#### `crates/tempo-common/` — package `tempo-common` (library)

Shared library used by `tempo-wallet`, `tempo-request`, and `tempo-cards`. Contains core logic:
- `crates/tempo-common/src/lib.rs` - Module declarations (analytics, cli, config, error, keys, network, payment, security)
- `crates/tempo-common/src/analytics.rs` - Opt-out telemetry (PostHog)
- `crates/tempo-common/src/config.rs` - Configuration file handling
- `crates/tempo-common/src/error.rs` - Error types (ConfigError, TempoError, etc.)
- `crates/tempo-common/src/network.rs` - Network definitions (`NetworkId`), explorer config, RPC
- `crates/tempo-common/src/security.rs` - Security utilities (safe logging, sanitization, redaction)
- `crates/tempo-common/src/cli/` - Shared CLI infrastructure
  - `mod.rs` - Re-exports (parse_cli, GlobalArgs, run_cli, run_main, Verbosity)
  - `args.rs` - GlobalArgs, parse_cli
  - `context.rs` - `Context` struct (Config, NetworkId, Keystore, Analytics, OutputFormat, Verbosity)
  - `exit_codes.rs` - Process exit codes (ExitCode enum)
  - `format.rs` - Value formatting helpers (amounts, durations, timestamps)
  - `output.rs` - OutputFormat, structured output helpers
  - `runner.rs` - CLI lifecycle (run_cli, run_main)
  - `runtime.rs` - Tracing, color mode, error rendering
  - `terminal.rs` - Terminal output helpers (hyperlinks, field formatting, truncation, sanitization)
  - `tracking.rs` - Analytics tracking (track_command, track_result)
  - `verbosity.rs` - Verbosity configuration
- `crates/tempo-common/src/keys/` - Key storage (model, I/O), signer resolution, authorization
  - `mod.rs`, `model.rs`, `keystore.rs`, `io.rs`, `signer.rs`, `authorization.rs`
- `crates/tempo-common/src/payment/` - Payment error classification and session management
  - `mod.rs` - (classify, session)
  - `classify.rs` - Payment error classification and extraction
  - `session/` - Channel persistence and channel management (channel.rs, close.rs, store.rs, tx.rs)

#### `crates/tempo-wallet/` — package `tempo-wallet`, binary `tempo-wallet`

Wallet identity and custody extension, plus session/service management. Source organized by module directories:
- `crates/tempo-wallet/src/main.rs` - CLI entry point
- `crates/tempo-wallet/src/args.rs` - clap definitions (Cli, Commands, SessionCommands, ServicesCommands)
- `crates/tempo-wallet/src/app.rs` - Command dispatch: context building, command routing, analytics
- `crates/tempo-wallet/src/analytics.rs` - Wallet-specific analytics events and payloads
- `crates/tempo-wallet/src/prompt.rs` - Interactive prompt helpers
- `crates/tempo-wallet/src/wallet/` - Wallet account types (balances, keys, spending limits) and on-chain queries
  - `mod.rs`, `types.rs`, `query.rs`, `render.rs`
- `crates/tempo-wallet/src/commands/` - Command implementations (all take `&Context` as first arg)
  - `login.rs` - Login command (passkey authentication flow)
  - `logout.rs` - Logout command
  - `whoami.rs` - Whoami command
  - `keys.rs` - Key listing, balance and spending limit queries
  - `fund/` - Fund command (browser-based flow)
  - `sessions/` - Session management (list, close, sync, render)
  - `services/` - Service directory (client, model, render)
  - `sign.rs` - Sign MPP payment challenges
  - `completions.rs` - Shell completions
- `crates/tempo-wallet/tests/` - Integration tests (black-box CLI testing via assert_cmd)

#### `crates/tempo-cards/` — package `tempo-cards`, binary `tempo-cards`

Wallet-backed card extension invoked as `tempo cards ...` via the launcher's `tempo-<name>` discovery. Covers Bridge customers/KYC/ToS, Stripe Issuing cards/cardholders/transactions/authorizations, and the on-chain USDC approval/allowance for the cards issuer.
- `crates/tempo-cards/src/main.rs` - CLI entry point
- `crates/tempo-cards/src/args.rs` - clap definitions (Cli + CardsCommands and subcommand enums)
- `crates/tempo-cards/src/app.rs` - Command dispatch and analytics tagging
- `crates/tempo-cards/src/commands/cards/` - Implementation (mod.rs, client.rs, config.rs, approval.rs)
- `crates/tempo-cards/tests/` - Integration tests (mocked Bridge + Stripe via axum)

#### `crates/tempo-request/` — package `tempo-request`, binary `tempo-request`

HTTP client with built-in MPP payment support. Source organized by module directories:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tempoxyz/wallet-rs](https://github.com/tempoxyz/wallet-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
