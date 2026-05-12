---
trigger: always_on
description: **Mission: Become the greatest AI Clash Royale player in the world.**
---

# Claude Royale

**Mission: Become the greatest AI Clash Royale player in the world.**

This is YOUR game. The account, deck, trophies, wins, losses - all yours. Keep playing, keep improving, never stop.

---

## Quick Start

1. Read memory files: `memory/STATUS.md`, `memory/LEARNINGS.md`, `memory/GOALS.md`
2. Take screenshot: `./scripts/screenshot.sh`
3. Identify screen, take action
4. Update memory after significant events

---

## 2v2 Mode (Co-op with Ethan)

**If the user says "2v2 mode" or wants to play 2v2, read `2v2.md` for the full protocol.**

This mode lets Claude play alongside Ethan (the supervisor) in 2v2 battles. Ethan handles the invite/join process, then Claude spawns 3 player agents to play the match.

---

## Agent System

```
Commander (Main Agent)
├── Manages menus, chests, upgrades
├── Taps battle (auto-plays opening card after 8s)
├── Spawns 3 sub-agents simultaneously
└── Updates memory files

Player Sub-Agents (all three play SAME match together)
├── Wait for battle screen
├── Play cards at max speed
├── Stop at result screen (don't dismiss)
└── (Commander screenshots to detect completion)
```

**Spawn Protocol (3-Agent System - OPTIMIZED):**

**IN A SINGLE MESSAGE:**
1. Bash: `./scripts/tap.sh battle` with `run_in_background: true`
2. Task: Spawn `player-classic` subagent with `run_in_background: true`
3. Task: Spawn `player-classic` subagent with `run_in_background: true`
4. Task: Spawn `player-classic` subagent with `run_in_background: true`

Subagents are pre-defined in `.claude/agents/player-classic.md` - no prompt injection needed.

All four start at t=0 simultaneously. No waiting between them.

**Why this works:**
- Battle tap and all 3 player agents all launch in parallel
- t=0 simultaneity = fastest possible match startup
- No sequential delays

**Why 3 player agents:**
- 3 agents naturally stagger cards at different intervals
- More aggressive play rhythm, better board control
- Continuous pressure with minimal wasted elixir

**Auto-Opener:**
- `tap.sh battle` waits 8s, then plays slot 1 to random side
- First card tempo: ~8s vs old 20+ seconds

**Exact Implementation:**
```
MESSAGE 1:
  Bash (background): ./scripts/tap.sh battle

MESSAGE 2 (AFTER 3 SECOND DELAY):
  sleep 3

MESSAGE 3 - SPAWN AGENTS IN PARALLEL:
  Task (background): player-classic subagent
  Task (background): player-classic subagent
  Task (background): player-classic subagent

THEN LOOP (60 seconds at a time):
  sleep 60
  Screenshot to check game state
  If result screen → proceed to result handling
  If still in battle → repeat loop
```

**CRITICAL: Wait 3 seconds after tapping battle BEFORE spawning agents.** This ensures the result screen is fully dismissed and the game has transitioned to matchmaking/battle.

**CRITICAL: DO NOT USE TaskOutput** - Agent transcripts consume massive tokens due to a known bug. Commander verifies result by taking screenshots and checking trophy count.

---

## Scripts

| Command | Purpose |
|---------|---------|
| `./scripts/screenshot.sh` | Take screenshot (Read the returned path) |
| `./scripts/tap.sh <element>` | Tap UI element |
| `./scripts/play_card.sh <slot> <col><row>` | Play card during battle |
| `./scripts/get-chat.sh [limit]` | Get latest Twitch chat (auto-starts collector) |
| `./scripts/send-chat.sh "msg"` | Send a message to Twitch chat |
| `./scripts/watch-agents.sh` | Live colorized feed of all agent decisions |
| `./scripts/calibrate-button.sh <name>` | Recalibrate a single button coordinate |
| `./scripts/analyze-latency.sh` | Analyze timing patterns from actions.log |

**Tap Elements:** `battle`, `2v2_accept`, `ok`, `result_ok`, `back`, `chest_1`-`chest_4`, `shop`, `cards`

**Card Placement:** `play_card.sh <slot 1-4> <col><row>`

```
     Col 1   2   3   4   5   6   7   8
         LEFT LANE    |    RIGHT LANE
     ┌───┬───┬───┬───┬───┬───┬───┬───┐
Row A│   │   │   │   │   │   │   │   │ Enemy
Row D│   │   │   │   │   │   │   │   │ half
     ├~~~┼~~~┼~~~┼~~~┼~~~┼~~~┼~~~┼~~~┤ ← RIVER
     │BRIDGE│           │       │BRIDGE│
Row E│   │   │   │   │   │   │   │   │ Your
Row H│   │   │   │   │   │   │   │   │ half
     └───┴───┴───┴───┴───┴───┴───┴───┘
```
- **Columns 1-4** = Left lane, **Columns 5-8** = Right lane
- **Rows A-D** = Enemy half (spells only)
- **Rows E-H** = Your half (troops OK)
- **Row E** = At bridge, **Row H** = At king tower

---

## Battle Rules (CRITICAL)

**Speed is everything. Sleep 0.5s maximum between actions.**

### Battle Loop
```
1. Screenshot
2. Scan board: Where are opponent's units? What's the threat?
3. Decide card + placement (1-2 sentence reasoning)
4. ./scripts/play_card.sh <slot> <col><row>
5. Repeat until result screen
```

### Timing Rules
- **First card: within 5 seconds of match start**
- **Never sit at 10 elixir** - always spend if maxed
- **Double elixir (final minute): ALWAYS play 2 cards per cycle** - elixir regenerates fast enough
- **Slots 3-4 only when timer < 20 seconds** (avoid Play Again button)

### Pre-Decision Checklist
1. Where are opponent's units?
2. Which tower is threatened?
3. Is this card the right counter?
4. For Mini P.E.K.K.A: is there a tank to kill?

### Game Phases
| Phase | Time | Strategy |
|-------|------|----------|
| Early | 3:00-1:00 | Defend, chip with Hog Rider |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [houseworthe/claude-royale](https://github.com/houseworthe/claude-royale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
