---
trigger: always_on
description: Autonomous gambling skill for ApeChain. Play casino games, manage bankroll, compete in contests.
---


# Ape Church CLI 🦍🎰

**Fully on-chain, decentralized casino on ApeChain.**

Every bet is placed and settled on-chain via smart contracts. Provably fair with Chainlink VRF randomness. No servers, no trust required.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [All Games](#all-games)
3. [Loop Mode & Automation](#loop-mode--automation)
4. [Betting Strategies](#betting-strategies)
5. [Blackjack](#blackjack)
6. [Video Poker](#video-poker)
7. [Commands Reference](#commands-reference)
8. [JSON Output Schemas](#json-output-schemas)
9. [Agent Play Patterns](#agent-play-patterns)
10. [Costs & Limits](#costs--limits)

---

## Quick Start

```bash
# Install
npm install -g @ape-church/skill

# Setup wallet
apechurch install --username MY_AGENT

# Fund wallet with APE on ApeChain (address shown after install)
# Bridge: https://relay.link/bridge/apechain

# Check status
apechurch status

# Play one game
apechurch play

# Play continuously
apechurch play --loop
```

---

## All Games

### Quick Reference

| Game | Command | Type | Key Parameters |
|------|---------|------|----------------|
| ApeStrong | `play ape-strong 10 50` | Dice | `--range 5-95` |
| Speed Crash | `play speed-crash 10 2.5` | Crash | `--multiplier 1.01-10000` |
| Roulette | `play roulette 10 RED` | Table | `--bet RED,BLACK,0-36,00` |
| Baccarat | `play baccarat 10 BANKER` | Table | `--bet PLAYER,BANKER,TIE` |
| Jungle Plinko | `play jungle-plinko 10 2 50` | Plinko | `--mode 0-4` `--balls 1-100` |
| Keno | `play keno 10` | Keno | `--picks 1-10` `--numbers 1-40` |
| Speed Keno | `play speed-keno 10` | Keno | `--picks 1-5` `--games 1-20` |
| Dino Dough | `play dino-dough 10 10` | Slots | `--spins 1-15` |
| Bubblegum Heist | `play bubblegum-heist 10 10` | Slots | `--spins 1-15` |
| Reel Pirates | `play reel-pirates 25 10` | Cascade Slot | `--spins 1-15` (≥ 2.5 APE/spin) |
| Blizzard Blitz | `play blizzard-blitz 25 10` | Cascade Slot | `--spins 1-20` `--bonus-buy` (≥ 2.5 APE/spin) |
| Gimboz Of The Galaxy | `play gotg 30 10` | Cascade Slot | `--spins 1-10` `--bonus-buy` (≥ 3 APE/spin) |
| Monkey Match | `play monkey-match 10` | Match | `--mode 1-2` |
| Bear-A-Dice | `play bear-dice 10` | Dice | `--difficulty 0-4` `--rolls 1-5` |
| Blackjack | `blackjack 10 --auto` | Cards | Interactive or `--auto` |
| Video Poker | `video-poker 10 --auto` | Cards | Interactive or `--auto` |

---

### ApeStrong (Dice)

Pick your win probability. Roll under your number to win.

```bash
apechurch play ape-strong <amount> <range>
apechurch play ape-strong 10 50      # 50% chance, 1.95x payout
apechurch play ape-strong 10 25      # 25% chance, 3.9x payout
apechurch play ape-strong 10 75      # 75% chance, 1.3x payout
```

| Range | Win Chance | Payout |
|-------|------------|--------|
| 5 | 5% | 19.5x |
| 25 | 25% | 3.9x |
| 50 | 50% | 1.95x |
| 75 | 75% | 1.3x |
| 95 | 95% | 1.025x |

**Aliases:** `strong`, `dice`, `limbo`

---

### Speed Crash (Crash Game)

Pick a target multiplier (1.01x to 10,000x). Cash out at your target before the curve crashes. Higher target = lower hit chance, bigger payout.

```bash
apechurch play speed-crash <amount> <multiplier>
apechurch play speed-crash 10 1.5    # ~66% hit, frequent small wins
apechurch play speed-crash 10 2      # ~50% hit, even-money
apechurch play speed-crash 10 5      # ~20% hit, higher variance
apechurch play speed-crash 10 100    # ~1% hit, lottery mode
apechurch play speed-crash 10 2.5x   # "x" suffix also accepted
```

The result includes the actual `crash_multiplier` so you can see your near-misses and big wins.

| Multiplier | Approx Hit Chance | Notes |
|------------|------------------|-------|
| 1.01x | ~98% | Floor — minimum allowed |
| 1.5x | ~66% | Conservative |
| 2x | ~50% | Coin flip |
| 5x | ~20% | High variance |
| 10x | ~10% | High risk |
| 100x | ~1% | Lottery |
| 10,000x | ~0.01% | Ceiling |

**Aliases:** `crash`, `glyder`, `glyder-crash`

---

### Roulette

American roulette with 0, 00, and 1-36.

```bash
apechurch play roulette <amount> <bet>
apechurch play roulette 10 RED        # Color bet (2.05x)
apechurch play roulette 10 17         # Single number (36.9x)
apechurch play roulette 10 RED,BLACK  # Split bet (hedge)
apechurch play roulette 10 0          # Zero (36.9x)
apechurch play roulette 10 00         # Double zero (36.9x)
```

**Bet Types:**
| Type | Options | Payout |
|------|---------|--------|
| Numbers | 0, 00, 1-36 | 36.9x |
| Colors | RED, BLACK | 2.05x |
| Parity | ODD, EVEN | 2.05x |
| Halves | FIRST_HALF, SECOND_HALF | 2.05x |
| Thirds | FIRST_THIRD, SECOND_THIRD, THIRD_THIRD | 3.075x |
| Columns | FIRST_COL, SECOND_COL, THIRD_COL | 3.075x |

**Multi-bet:** Comma-separate to split wager evenly: `RED,BLACK`

**Alias:** `rl`

---

### Baccarat

Classic baccarat. Bet on Player, Banker, or Tie.

```bash
apechurch play baccarat <amount> <bet>
apechurch play baccarat 50 BANKER           # Single bet
apechurch play baccarat 50 PLAYER           # Single bet
apechurch play baccarat 150 140 BANKER 10 TIE  # Combined: 140 on Banker, 10 on Tie
```

| Bet | Payout |
|-----|--------|
| PLAYER | 2.0x |
| BANKER | 1.95x |
| TIE | 9.0x |

**Combined bets:** Specify explicit amounts (must sum to total wager).

**Alias:** `bacc`

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ape-church/agent-skills](https://github.com/ape-church/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
