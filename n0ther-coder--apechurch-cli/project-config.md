---
trigger: always_on
description: Autonomous gambling skill for ApeChain. Play casino games, manage bankroll, compete in contests.
---


# Ape Church CLI 🦍🎰

> Summary: Agent-facing manual for the Ape Church CLI skill bundle. Documents commands, game syntax, automation patterns, JSON schemas, bankroll constraints, and recommended operating flows.

**Fully on-chain, decentralized casino on ApeChain.**

Every bet is placed and settled on-chain via smart contracts. Provably fair with Chainlink VRF randomness. No servers, no trust required.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [History Download & Reporting](#history-download--reporting)
3. [All Games](#all-games)
4. [Loop Mode & Automation](#loop-mode--automation)
5. [Betting Strategies](#betting-strategies)
6. [Blackjack](#blackjack-)
7. [Video Poker](#video-poker)
8. [Commands Reference](#commands-reference)
9. [JSON Output Schemas](#json-output-schemas)
10. [Agent Play Patterns](#agent-play-patterns)
11. [Costs & Limits](#costs--limits)

---

## Quick Start

```bash
# Install
npm install -g @n0ther/apechurch-cli

# Optional for non-interactive local signing
export APECHURCH_CLI_PASS=your-local-password

# Optional to override the username/profile API
export APECHURCH_CLI_PROFILE_URL=https://www.ape.church/api/profile

# Fresh install/import prompts securely for the private key
apechurch-cli install --username MY_AGENT

# Fund wallet with APE on ApeChain (address shown after install)
# Bridge: https://relay.link/bridge/apechain

# Check status
apechurch-cli status

# Download wallet history from chain data
apechurch-cli wallet download

# Read cached history stats
apechurch-cli history --stats

# Play one random game/config automatically
apechurch-cli play --auto

# Play continuously
apechurch-cli play --loop
```

On a fresh install/reinstall, `apechurch-cli install` prompts securely for the private key with hidden input unless `~/.apechurch-cli/wallets/current.json` already points to an encrypted `wallets/<address>.json` entry. `APECHURCH_CLI_PK` remains an optional non-interactive fallback, `APECHURCH_CLI_PASS` is required for non-interactive install/signing, and `APECHURCH_CLI_PROFILE_URL` overrides the default username/profile API.

---

## History Download & Reporting

Use `wallet download` to reconstruct supported gaming history directly from ApeChain into a local per-wallet cache, then read it with `history`.

If `[address]` is omitted, both commands use the local wallet address.

```bash
# Download the local wallet history
apechurch-cli wallet download

# Download a specific address
apechurch-cli wallet download 0x1234...abcd

# Scan only a recent block range
apechurch-cli wallet download 0x1234...abcd --from-block 35000000 --to-block 35300000

# Rebuild the local history file from genesis
apechurch-cli wallet download 0x1234...abcd --from-block 0

# JSON output for automation
apechurch-cli wallet download 0x1234...abcd --json

# Read saved history and stats
apechurch-cli history 0x1234...abcd

# Show more than the default 10 cached games
apechurch-cli history 0x1234...abcd --limit 25

# Show every cached game
apechurch-cli history 0x1234...abcd --all

# Stats only
apechurch-cli history 0x1234...abcd --stats

# Stats split by game
apechurch-cli history 0x1234...abcd --breakdown

# Weekly wAPE wagered totals with play breakdowns
apechurch-cli history 0x1234...abcd --leaderboard

# Refresh before reading
apechurch-cli history 0x1234...abcd --refresh

# Merge a full-range refresh before reading
apechurch-cli history 0x1234...abcd --refresh --from-block 0
```

Sync and cache behavior:

- `wallet download` is incremental by default. Without `--from-block`, it resumes from `last_synced_block + 1`.
- Use `wallet download --from-block 0` to rebuild the local history file from scratch, or pass an explicit historical range to fill older blocks.
- Explicit backfills and `history --refresh` are merged into the local file and deduplicated by `contract + game_id`.
- `history --refresh` runs the same sync path before reading the local file, but it does not clear cached records first.
- `history` shows `👀 Recent Games` plus `📜 History Stats` by default. `--stats` suppresses the game list, while `--breakdown` appends the same stats split by game.
- Standard `history` output also includes a compact `🎮 Game Status` section with per-game `played`, `net`, `win rate`, `RTP`, and local `unfinished` counts when available.

### Report Fields

| Field | Meaning |
|------|---------|
| `🎰 Games` | Economically synced games included in totals |
| `💸 Contract fees paid` | Contract-side fees effectively paid by the wallet |
| `⛽️ Gas paid` | Network gas effectively paid by the wallet |
| `Net result` | `payout - wager - contract fees - gas` |
| `✌️ Win rate` | Wins divided by economically synced games |
| `🎲 RTP` | `total payout / total wagered` |
| `🎟️  APE Wagered (wAPE)` | Current on-chain balance / total APE wagered by synced games |
| `🧮 Gimbo Points (GP)` | Current on-chain balance / total received from synced games; every `10,000 GP` equals `1 Level` |

### Wallet Download Options

| Option | Description |
|------|---------|
| `wallet download --from-block <n>` | Start block for the sync; `--from-block 0` rebuilds the history file |
| `wallet download --to-block <n>` | End block for the sync (default: latest block) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [n0ther-coder/apechurch-cli](https://github.com/n0ther-coder/apechurch-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
