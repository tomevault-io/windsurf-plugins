---
trigger: always_on
description: An MCP server connecting to a live Civilization VI game via FireTuner. You can read full game state and issue commands. All commands respect game rules.
---

# Civ 6 MCP — Agent Reference

An MCP server connecting to a live Civilization VI game via FireTuner. You can read full game state and issue commands. All commands respect game rules.

**You only know what you explicitly query.** A human player passively absorbs the score ticker, religion lens, unit health bars — you have none of that. Information you don't ask for simply doesn't enter your world model. The patterns below exist to compensate for this.

`end_turn` now runs **empire warnings** automatically — alerts for loyalty crises, idle trade routes, gold deficits, resource caps, scoreboard position, and military imbalance. These compensate for the most common blind spots, but don't replace periodic deep checks (victory progress, religion spread, diplomacy).

## Coordinate System

**Hex grid: (X, Y) where higher Y = visually south.**
- Y increases → south (down). Y decreases → north (up).
- X increases → east. X decreases → west.
- Moving from (9,24) to (9,26) is **south**, not north.

## Game Start

Before your first turn:
1. Read your civ's unique abilities, units, and buildings — what is this civ designed to do?
2. Identify the tech/civic that unlocks your unique unit; plan a research path to reach it.
3. Form a working hypothesis for a victory path. Hold it loosely — geography and rivals will clarify things through the Classical era.

Early choices compound. Each decision shapes what's available 20, 40, 60 turns later. A scout reveals the map early; a defensive unit lets your settlers move safely; more cities mean more districts which mean more everything. Religious civs often benefit from Holy Site infrastructure before the Great Prophet pool fills. What you don't build early, you pay for later.

## Turn Loop

Each turn in order:
1. `get_game_overview` — turn, yields, research, score, era score, difficulty. If resuming after context compaction, call `get_diary` first.
2. `get_units` — positions, HP, moves, charges, nearby threats
3. `get_map_area` around cities/units — terrain, resources, enemy units
4. Move/action each unit
5. `get_cities` — queues, growth, pillaged districts
6. `get_district_advisor` if placing a new district
7. `set_city_production` / `set_research` if needed
8. Run **Strategic Checkpoints** if it's time
9. `end_turn`

## Diary

The diary is your persistent memory across sessions. When context compacts or you return to a game, `get_diary` is how you reconstruct where you were and why you made the decisions you did. Entries with specific details — unit names, coordinates, yield numbers, reasoning — are far more useful to your future self than brief summaries.

Reflections are recorded **before** AI processing begins — write what YOU observed and did this turn. Anything that surfaces after `end_turn` (a diplomacy proposal, AI units entering your territory, events in the turn result) belongs in the **next** turn's diary, not this one.

Five reflection fields each turn (all required, non-empty):
- **tactical**: What happened — specific units, tiles, outcomes.
- **strategic**: Standings vs rivals — yields, city count, victory path viability with numbers.
- **tooling**: Tool issues observed, or "No issues".
- **planning**: Concrete actions for the next 5-10 turns — specific builds, moves, research targets with turn estimates.
- **hypothesis**: Specific predictions — attack timing, milestone turns, biggest risks.

## Strategic Checkpoints

Periodic checks worth doing regularly. The game doesn't surface most of this proactively.

### Around every 10 turns:
- `get_empire_resources` — unimproved luxuries and nearby strategics
- Surplus luxuries: duplicates beyond 1 copy provide zero amenity benefit. Trade them via `propose_trade` for GPT, strategic resources, or luxury types you don't own (each new type = +1 amenity to 4 cities). Even 5 GPT per surplus luxury adds up over 30 turns. Use `mode="test"` to check what the AI will accept before sending.
- Gold/faith balance: if either is accumulating with no plan, spend it — `purchase_item`, `purchase_tile`, `patronize_great_person`
- City count vs time in game — if expansion is behind, a settler tends to be the highest-leverage production choice
- `get_trade_routes` — check for idle routes; idle routes are free yields going uncollected
- Government tier — `change_government` when a new tier unlocks (free the first time)
- Era score vs thresholds — shown in `get_game_overview`; a Dark Age is recoverable but costly
- Great People — `get_great_people`; rivals will recruit what you don't

### Around every 20 turns:
- `get_diplomacy` — delegations to new civs, friendships with Friendly civs, alliances if eligible
- `get_victory_progress` — check all 6 victory types, not just your own path
- `get_religion_spread` — religious victory is invisible without active checking; a rival with majority in most civs is a serious threat

### Around every 30 turns:
- `get_strategic_map` — fog per city + unclaimed resources
- `get_global_settle_advisor` — best remaining settle sites
- Wonder scan: `get_city_production` in your best city — wonders that align with your victory path are worth considering

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lmwilki/civ6-mcp](https://github.com/lmwilki/civ6-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
