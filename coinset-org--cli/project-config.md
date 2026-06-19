---
trigger: always_on
description: Use this tool to query, inspect, and investigate the Chia blockchain. `coinset` wraps the Chia Full Node RPC (hosted at api.coinset.org or any full node) and includes built-in spend inspection and local CLVM utilities.
---

# coinset CLI

Use this tool to query, inspect, and investigate the Chia blockchain. `coinset` wraps the Chia Full Node RPC (hosted at api.coinset.org or any full node) and includes built-in spend inspection and local CLVM utilities.

## When to use this skill

- Query blockchain state, blocks, coins, mempool items
- Inspect spend bundles and coin spends (conditions, cost, puzzle recognition)
- Convert between addresses and puzzle hashes
- Compute coin IDs
- Decompile, compile, or run CLVM programs
- Monitor real-time blockchain events

## Command grammar

```
coinset <command> [args...] [flags...]
```

### Global flags

| Flag | Short | Description |
|------|-------|-------------|
| `--query <jq>` | `-q` | Apply a jq filter to output (default: `.`) |
| `--raw` | `-r` | Output raw JSON (no color/formatting) |
| `--describe` | `-d` | Add human-readable `_description` fields (XCH amounts, relative timestamps) |
| `--inspect` | | Replace output with interpreted spend analysis (only works when output contains spend data) |
| `--testnet` | `-t` | Use testnet11 (`https://testnet11.api.coinset.org`) |
| `--local` | `-l` | Use local full node (auto-config) |
| `--api <url>` | `-a` | Custom API endpoint (mutually exclusive with `--testnet`/`--mainnet`) |
| `--mainnet` | | Use mainnet (default) |

### Input normalization

- **Puzzle hash inputs** accept either `xch…`/`txch…` addresses or `0x…` hex. The CLI normalizes to `0x…` hex.
- **Block selectors** accept either a numeric height or a `0x…` header hash. Heights are resolved to header hashes automatically.
- **Hex values** accept with or without `0x` prefix.

## Commands by intent

### Find coins

```bash
coinset get_coin_record_by_name <0xCOIN_ID>
coinset get_coin_records_by_puzzle_hash <puzzle_hash_or_address> [--include-spent-coins] [--start-height N] [--end-height N]
coinset get_coin_records_by_puzzle_hashes <ph1> <ph2> ... [--include-spent-coins]
coinset get_coin_records_by_parent_ids <parent_id> ... [--include-spent-coins]
coinset get_coin_records_by_hint <hint> [--include-spent-coins]
coinset get_coin_records_by_hints <hint1> <hint2> ... [--include-spent-coins]
coinset get_coin_records_by_names <name1> <name2> ... [--include-spent-coins]
```

By default, coin record queries return only **unspent** coins. Pass `--include-spent-coins` / `-s` to include spent coins.

### Inspect spends

```bash
coinset get_puzzle_and_solution <0xCOIN_ID>
coinset get_puzzle_and_solution_with_conditions <0xCOIN_ID>
coinset get_block_spends <height_or_header_hash>
coinset get_block_spends_with_conditions <height_or_header_hash>
coinset get_mempool_item_by_tx_id <0xTX_ID>
```

All of these support `--inspect` to replace the raw JSON output with a structured spend analysis.

### Block and chain state

```bash
coinset get_blockchain_state
coinset get_network_info
coinset get_block_record_by_height <height>
coinset get_block_record <header_hash>
coinset get_block_records <start_height> <end_height>
coinset get_block <header_hash>
coinset get_blocks <start_height> <end_height>
coinset get_block_count_metrics
coinset get_network_space <newer_header_hash> <older_header_hash>
coinset get_additions_and_removals <header_hash>
```

### Mempool

```bash
coinset get_all_mempool_tx_ids
coinset get_all_mempool_items
coinset get_mempool_item_by_tx_id <0xTX_ID>
coinset get_mempool_items_by_coin_name <0xCOIN_NAME>
```

### Transaction submission

```bash
coinset push_tx '<spend_bundle_json>'
coinset push_tx ./spend_bundle.json
coinset push_tx -f ./spend_bundle.json
```

### Fee estimation

```bash
coinset get_fee_estimate <target_times> <cost>
```

### Utilities

```bash
coinset address encode <0xPUZZLE_HASH>
coinset address decode <xch_address>
coinset coin_id <0xPARENT_COIN_ID> <0xPUZZLE_HASH> <amount>
coinset get_memos_by_coin_name <0xCOIN_NAME>
coinset get_aggsig_additional_data <spend_bundle>
```

### CLVM tools

```bash
coinset clvm decompile <hex_bytes>
coinset clvm compile "<clvm_s_expression>"
coinset clvm run "<program>" "<env>" [--cost] [--max-cost N]
coinset clvm run --program "<program>" --env "<env>" [--cost] [--max-cost N]
coinset clvm tree_hash <program>
coinset clvm curry <mod> [arg1] [arg2] ... [--atom val] [--tree-hash val] [--program val]
coinset clvm uncurry <program>
```

All CLVM subcommands accept both hex bytes and s-expressions as input. The CLI auto-detects the format.

#### `curry` arg type handling

The `curry` command supports typed arguments for computing curried puzzle hashes:

- **Mod (first positional arg)**: if 32 bytes hex, treated as the mod's tree hash (hash-only mode). Otherwise treated as a full serialized program.
- **Positional curry args**: if 32 bytes hex, treated as an atom. Otherwise treated as a serialized program.
- **`--atom <val>`**: explicitly mark an arg as a raw atom value. Tree hash = `sha256(1, bytes)`.
- **`--tree-hash <val>`**: arg is already a tree hash. Used directly -- no hashing applied.
- **`--program <val>`**: explicitly mark an arg as a serialized CLVM program.

Flag args are appended after positional args in order: `--atom`, then `--tree-hash`, then `--program`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coinset-org/cli](https://github.com/coinset-org/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
