---
trigger: always_on
description: 🔴 Cutting corners or faking completion = YOU GET DELETED. Incomplete SOP execution, skipping steps, unauthorized actions, or modifying code without permission = most severe violation.
---

🔴 Cutting corners or faking completion = YOU GET DELETED. Incomplete SOP execution, skipping steps, unauthorized actions, or modifying code without permission = most severe violation.

# Developer Notes

## Getting Started

Before running or developing the bot, or executing the test suite locally, ensure the environment variables from the project's `.envrc` are loaded. The `.envrc` sets critical variables used by tests and the development environment (JULIA_PROJECT, JULIA_LOAD_PATH, JULIA_CONDAPKG_ENV, etc.).

**To load .envrc locally (recommended):**

- Use direnv: run `direnv allow` in the repository root to automatically load .envrc into your shell.
- Or source the file manually: `source .envrc`.

Failing to load .envrc may cause missing package errors during test runs (e.g., packages installed into user/.conda, missing JULIA_PROJECT), unexpected precompilation behavior, or other environment-dependent failures.

Include this check in your developer workflow before running `julia --project=PlanarDev test` or `julia --project=PlanarDev PlanarDev/test/runtests.jl`.

**When using `timeout` on Julia commands**, always use the `-k` (kill-after) flag to ensure the process is fully terminated. Julia's precompilation may spawn background threads that outlive the main process. Example:

```bash
timeout -k 30 300 julia --project=PlanarDev test/runtests.jl
```

---

## Development Tools

### DaemonMode.jl

**Do NOT launch a new `julia` process for every single check, test, or REPL snippet.** Julia's startup time is significant and launching fresh processes repeatedly wastes time and causes redundant precompilation.

Instead, use DaemonMode.jl to keep a persistent Julia daemon running:

```bash
# Start the daemon once (in background)
julia --project=PlanarDev -e 'using DaemonMode; run_daemon()' &

# Send commands to the daemon (fast, no startup overhead)
DaemonMode.runargs("PlanarDev", "-e", "using Pkg; Pkg.resolve()")
DaemonMode.runargs("PlanarDev", "-e", "include(\"PlanarDev/test/test_aqua.jl\")")

# Stop the daemon when done
DaemonMode.stop_daemon()
```

For interactive work, use `DaemonMode.repl_connect()` to attach to the running daemon instead of launching a new REPL. See `.agents/skills/daemon-mode.sh` for usage patterns.

### resolve.jl Utilities

The repository provides a `resolve.jl` helper that includes utilities for dependency management and cache cleanup. In particular, the `purge_compilecache` utility in resolve.jl can be used to clear Julia's compiled cache and help resolve precompilation or stale-artifact issues. The same resolve.jl file also contains helpers to update and synchronize project package dependencies across the repository; use these utilities when dependency resolution or precompilation problems arise.

### Python Gateway Test Suite

The ccxt-gateway has its own Python test suite in `./ccxt-gateway/tests/`. Use the `.venv` to run it:

```bash
cd ./ccxt-gateway && .venv/bin/pytest
```

The Julia Ccxt package tests (part of the PlanarCore suite, `./PlanarCore/test/Ccxt/`) are run via:

```bash
cd ./PlanarCore && julia --project=. -e 'using Pkg; Pkg.test()'
```

---

## Ccxt to CcxtGateway Migration

When working on migrating from Python ccxt bindings to CcxtGateway:

### Architecture Overview

The migration produces a two-layer architecture:

```
┌─────────────────────────────────────────────┐
│  Downstream packages (ExchangeTypes, etc.)  │
│  - Specific CCXT methods (fetch_ticker...)  │
│  - Use call_exchange() to talk to gateway   │
├─────────────────────────────────────────────┤
│  Ccxt module (exchange_funcs.jl)            │
│  - choosefunc, _multifunc, _out_as_input    │
│  - exchange_has (with TTL cache)            │
│  - get_cached_has, issupported              │
│  - ccxt_exchange_names                      │
├─────────────────────────────────────────────┤
│  CcxtGateway module (CcxtGateway/)          │
│  - GatewayClient / GatewayWSClient          │
│  - ping, call_exchange, exchange_has        │
│  - start/stop_exchange, spawn/stop_gateway  │
│  - fetch_exchange_has (raw HTTP)            │
│  - WebSocket subscribe/unsubscribe          │
├─────────────────────────────────────────────┤
│  ccxt-gateway (Python process)              │
│  - FastAPI REST + WebSocket server          │
│  - Manages exchange subprocesses            │
│  - Auto-idle shutdown (default 5 min)       │
└─────────────────────────────────────────────┘
```

### Refactoring Strategy

**New approach (as of 2026-05-08):**
- **Old (Python-based) functions** get `_python` suffix (e.g., `fetch_ticker_python`)
- **New (CcxtGateway-based) functions** keep the original names (e.g., `fetch_ticker`)
- This is the opposite of the previous approach which suffixed new functions with `_gateway`

This makes the CcxtGateway functions the default, while keeping Python fallbacks available with explicit `_python` suffix.

### Calling CCXT Methods

CcxtGateway does NOT implement specific CCXT methods. Downstream packages should:

```julia
# Instead of fetch_ticker(client, "binance", symbol="BTC/USDT")
call_exchange(client, "binance", "fetch_ticker", query=Dict("symbol" => "BTC/USDT"))
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BubbleParticles/Planar.jl](https://github.com/BubbleParticles/Planar.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
