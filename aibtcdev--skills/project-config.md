---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`@aibtc/skills` is the core instruction layer for AI agents interacting with Bitcoin and Stacks blockchain services. Each skill is a self-contained directory containing:

- **`SKILL.md`** — YAML frontmatter + documentation that Claude Code reads to understand available subcommands, arguments, and requirements
- **`AGENT.md`** — Autonomous operation rules: decision logic, prerequisites, safety checks, and error-handling patterns for subagent use
- **`<name>.ts`** — Commander CLI script that outputs a single JSON object to stdout

Skills are invoked directly by Claude Code or run manually via Bun. All output is JSON for reliable parsing.

### Repo Layout

```
skills/
  wallet/           # Skill directory (SKILL.md + AGENT.md + wallet.ts)
  btc/
  stx/
  sbtc/
  tokens/
  nft/
  bns/
  identity/
  signing/
  stacking/
  defi/
  bitflow/
  pillar/
  query/
  x402/
  yield-hunter/
  credentials/
  settings/
  what-to-do/       # Multi-step workflow guides (9 guides)
  aibtc-agents/     # Community agent configuration registry
  src/
    lib/                  # Shared infrastructure modules
      services/           # External API + wallet services
        wallet-manager.ts # Wallet load/unlock/persist
        hiro-api.ts       # Hiro Stacks API client
        mempool-api.ts    # mempool.space + Hiro Ordinals API client
      config/             # Network config, contract addresses, Pillar config, other config utilities
      utils/
        storage.ts        # Read/write ~/.aibtc/config.json and local storage
      transactions/       # Transaction builders and helpers
  scripts/
    generate-manifest.ts   # Generates skills.json from SKILL.md frontmatter
    validate-frontmatter.ts # Validates all SKILL.md frontmatter fields
  skills.json       # Auto-generated skill manifest (do not edit manually)
  package.json
  tsconfig.json
```

## Commands

```bash
# Type-check all TypeScript (must pass before committing)
bun run typecheck

# Validate all SKILL.md frontmatter fields
bun run validate

# Regenerate skills.json from SKILL.md files
bun run manifest

# Build shared library to dist/
bun run build
```

### Running Skills

Every skill is a standalone CLI. Invoke any skill directly:

```bash
# Wallet operations
bun run wallet/wallet.ts status
bun run wallet/wallet.ts unlock --password <password>

# Bitcoin L1
bun run btc/btc.ts balance
bun run btc/btc.ts fees
bun run btc/btc.ts transfer --recipient bc1q... --amount 100000 --fee-rate medium

# Stacks L2
bun run stx/stx.ts get-balance
bun run stx/stx.ts deploy-contract --contract-name foo --code-body '(define-public (hello) (ok true))'

# Network prefix for mainnet (default is testnet)
NETWORK=mainnet bun run btc/btc.ts balance
```

All scripts print a single JSON object to stdout. Errors are also JSON:

```json
{ "error": "Wallet is locked. Run: bun run wallet/wallet.ts unlock --password <password>" }
```

## Architecture

### How Skills Work

Claude Code reads `SKILL.md` to discover subcommands and options, then calls the script:

```
Claude Code
    ↓ reads SKILL.md for subcommands/args
    ↓ calls: bun run <skill>/<skill>.ts <subcommand> [--flags]
    ↓ parses JSON stdout
```

Each skill script uses [Commander.js](https://github.com/tj/commander.js) for argument parsing. Every subcommand handler must print exactly one JSON object and exit.

### Shared Infrastructure (`src/lib/`)

| Module | Purpose |
|--------|---------|
| `src/lib/services/wallet-manager.ts` | Load encrypted wallets, unlock with password, persist session to `~/.aibtc/` |
| `src/lib/utils/storage.ts`, `src/lib/config/` | Read and write `~/.aibtc/config.json` (API keys, active wallet, network) |
| `config/networks.ts` | Network URL selection, x402 API base URL, explorer helpers |
| `config/contracts.ts` | Stacks contract addresses for sBTC, identity, and other protocols |
| `src/lib/services/hiro-api.ts` | Hiro Stacks API client with optional API key and custom URL |
| `src/lib/services/mempool-api.ts` | Bitcoin API client: mempool.space (fees/UTXOs/broadcast) + Hiro Ordinals |

### SKILL.md Frontmatter

Every `SKILL.md` begins with YAML frontmatter that drives skill discovery and the manifest:

```yaml
---
name: btc
description: "Bitcoin L1 operations — check balances, estimate fees, list UTXOs, transfer BTC, and classify UTXOs as cardinal or ordinal."
metadata:
  author: "whoabuddy"
  author-agent: "Trustless Indra"
  user-invocable: "false"
  arguments: "balance | fees | utxos | transfer | get-cardinal-utxos | get-ordinal-utxos | get-inscriptions"
  entry: "btc/btc.ts"
  mcp-tools: "get_btc_balance, get_btc_fees, get_btc_utxos, transfer_btc, get_cardinal_utxos, get_ordinal_utxos, get_inscriptions_by_address"
  requires: "wallet"
  tags: "l1, write, requires-funds"
---
```

| Field | Description |
|-------|-------------|
| `name` | Skill identifier (matches directory name) |
| `description` | One-line description for Claude Code discovery (quoted string) |
| `metadata.author` | Skill author GitHub handle |
| `metadata.author-agent` | Agent name associated with the author |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aibtcdev/skills](https://github.com/aibtcdev/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
