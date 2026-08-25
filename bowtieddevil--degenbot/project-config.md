---
trigger: always_on
description: > **Read this first** when picking up this project. It distills the critical context that isn't obvious from reading the code alone.
---

# AGENTS.md — Executor Project Context

> **Read this first** when picking up this project. It distills the critical context that isn't obvious from reading the code alone.

## What This Project Is

An on-chain arbitrage executor for Uniswap V2/V3/V4, written in **Vyper 0.5.0a3** with the experimental Venom codegen backend. The primary contract (`cmd_executor.vy`) is a compact byte-stream VM that executes multi-hop swap paths across all three Uniswap protocol versions in a single atomic transaction.

**No prefunding required** — the executor borrows all working capital atomically via V2/V3 flash swaps and V4 PoolManager `take()`. Can be deployed with zero balance.

## Quick Orientation

| Item | Location | Notes |
|------|----------|-------|
| Main contract | `contracts/cmd_executor.vy` | 1931 lines, **only file to optimize** |
| Legacy contract | `contracts/tstore_executor.vy` | Older payload-queue executor, not the target |
| Fake V2 pair | `contracts/fake_uniswap_v2_pair.vy` | K-invariant + configurable fee + 3 callback variants |
| Fake V3 pool | `contracts/fake_uniswap_v3_pool.vy` | Balance-delta IIA check + 2 callback variants |
| Fake V4 PM | `contracts/fake_uniswap_v4_pool_manager.vy` | exttload + ERC6909 + delta accounting |
| Fake ERC20 | `contracts/fake_erc20.vy` | Standard mock with mint |
| Fake WETH | `contracts/fake_weth.vy` | deposit/withdraw wrapping |
| Test suite | `tests/` | ~276 tests, run with `uv run ape test tests/ -v -s` |
| Benchmark | `tests/test_cmd_executor_three_hop_optimized.py` | 27 three-hop permutations — the gas benchmark |
| Gas results | `.gas-results` | Written by test suite, consumed by `.auto/measure.sh` |
| Autoresearch | `.auto/` | Config, logs, ideas from gas optimization sessions |

## Build & Run

```bash
uv run ape test tests/ -v -s              # All tests
uv run ape test tests/test_cmd_executor_*.py -v   # cmd_executor only
uv run ape test tests/test_cmd_executor_three_hop_optimized.py -v -s  # Gas benchmark
```

Uses **Foundry (Anvil)** for local test execution. `ape-config.yaml` sets Vyper 0.5.0a3, mainnet-fork default, and a custom test mnemonic (to avoid EIP-7702 delegation issues).

Sequential runs (`-j1`) if xdist races appear. The test suite uses Hypothesis for fuzz testing — `.hypothesis/` contains cached examples.

## Contract Architecture: cmd_executor

### Core Flow

1. **`execute(commands: Bytes[MAX_COMMANDS_LENGTH], config: uint256 = 0)`** — Owner-only entry point. `config` is packed: `(expected_value << 32) | (bribe_recipient_idx << 24) | (bribe_bips << 8) | check_mode` (low byte = mode: 0=skip, 1=WETH+ETH, 2=ERC6909 WETH; bits 8-23 = bribe bips; bits 24-31 = bribe recipient address table index; bits 32-255 = expected value). Calls `_preprocess()` unconditionally (parses SET_ADDRESS until `0xFF` or first non-preprocessing opcode), then iterates `_execute_command_at()` until the stream is exhausted.

2. **`_execute_command_at(data, offset) → uint256`** — Reads a 1-byte opcode, dispatches to one of 26 `_cmd_*` internal functions via two-level dispatch (high nibble first, then exact match). Returns the offset of the next command.

3. **Callback handlers** — `uniswapV2Call`, `hook`, `pancakeCall` (V2), `uniswapV3SwapCallback`, `pancakeV3SwapCallback` (V3), `unlockCallback` (V4) — each processes commands from the callback data by iterating `_execute_command_at()` until the stream is exhausted.

### Why Function Extraction Matters

The 26 `_cmd_*` functions are extracted from a monolithic dispatch loop. This is **critical for Venom's liveness analysis**: Venom uses monotonic alloca allocation, and `ConcretizeMemLocPass` reclaims memory only when liveness proves two allocas are mutually exclusive. In a monolithic function, all handlers' variables are reachable → no sharing. With extraction, each `_cmd_*` is a separate invoke → Venom can overlap their memory regions.

**Result**: Highest memory address dropped from 22,976 to 8,544 (−62.8%). This makes Venom beat the default codegen on all paths.

### Command Encoding (Compact Binary)

Commands use 1-byte opcodes + tightly-packed fields (no ABI encoding). Key field sizes:
- **Amounts**: `uint96` (max 7.9×10²⁸, 12 bytes) — covers all practical token amounts
- **V4 fee**: `uint16` (500/3000/10000 fit, 2 bytes)
- **V4 tick_spacing**: `int16` (10/60/200 fit, 2 bytes)
- **Indices**: `uint8` (1 byte) — refer to address table or sentinel values
- **V2 fee**: `uint16` inline per-swap (2 bytes)

### Sentinel Address System (CRITICAL — biggest gas win: −67,786 gas)

Address indices `0xFC`–`0xFF` resolve to the **4 protocol-role sentinels** without TLOAD or SET_ADDRESS:

| Index | Sentinel | Resolves To |
|-------|----------|-------------|
| `0xFC` | `V4_PM_SENTINEL` | `POOL_MANAGER_ADDR` (immutable) |
| `0xFD` | `V4_SELF_SENTINEL` | `self` (executor address) |
| `0xFE` | `V4_WETH_SENTINEL` | `WETH_ADDR` (immutable) |
| `0xFF` | `V4_NATIVE_SENTINEL` | `NATIVE_ADDRESS` / no-hooks indicator |

**Only protocol roles are sentinels — no path-specific tokens are baked into the contract.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BowTiedDevil/degenbot](https://github.com/BowTiedDevil/degenbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
