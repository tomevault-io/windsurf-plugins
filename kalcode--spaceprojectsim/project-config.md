---
trigger: always_on
description: Context for Claude Code (claude.ai/code) when working in the `simulator-rust/` project.
---

# CLAUDE.md

Context for Claude Code (claude.ai/code) when working in the `simulator-rust/` project.

## What this is

Rust space-economy simulator + native Bevy client. The simulation engine
(`sim_core`) is a port of an Elixir/Phoenix prototype — moved off BEAM
because its scheduler performed poorly on Windows gaming PCs. The client
(`client_bevy`) embeds the engine directly and renders + inspects it in
one process, no socket boundary.

Ships at ~282 ships + ~95 facilities + 27 pops + 60 bodies + 8 factions
(~485 agents) today, designed to scale to 100k+. Single native binary,
no runtime dependencies. The Godot client referenced in older history
sections is archived under `client/` as a parity reference and is no
longer the canonical front end — every new feature lands in
`client_bevy`.

See [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) for mermaid
diagrams of the tick loop / AI stack / economic feedback loop /
runtime ownership / state machine, and [`plan.md`](./plan.md) for
the bundled improvement roadmap. `git log --oneline` is the source
of truth for recent activity.

## Status

**Phases 1–20c + Bundles 1–2 + GOAP arc + Bundle-4 tranche (S/H/G/Q/K) complete.**
190 tests passing, strict verify clean, runtime p50 ≈ 10–20 ms /
p99 ≈ 30–50 ms at 485 agents with WORLD_SPEED=4 (125 ms budget).
See [`plan.md`](./plan.md) for the source-of-truth improvement
roadmap.

**Bundle 1 (starter wins):** price memory decay on `best_trade_route`
(freshness half-life 1500 ticks), shortage-scaled delivery contract
quantity, personality drift on delivery success + rest-timeout,
facility + population LOD gating so off-screen agents skip heavy
sub-phases.

**Bundle 2 (AI planner):**
- 2.1 — **Unified utility scorer** replaces the priority waterfall.
  Every goal scored uniformly with tier weights (bio 100 /
  housekeeping 40 / strategic 3 / explore 6 / idle floor); the
  waterfall emerges from scores rather than hardcoded bypass.
  `FUEL_URGENCY_ONSET = 0.45` softened from the old 0.25 cliff.
- 2.2 — **Mid-action replan.** `ShipAI` gains `current_goal_score`
  + `last_plan_tick`; every `REPLAN_CHECK_INTERVAL_TICKS=20` a
  running plan may be interrupted if an alternative scores
  `× REPLAN_SCORE_RATIO=1.30`+ better, gated by
  `REPLAN_MIN_COMMIT_TICKS=40` hysteresis. Biological + retrofit
  + combined-goal plans are replan-protected.

**GOAP arc (replaces every hand-authored BT builder):**
- MVP — forward planner `sim_core/src/ai/planner.rs` with abstract
  `WorldState` (symbolic `Loc` + docked/fuel_bucket/cargo_loaded/
  delivered/sold/retrofitted/rested flags), `Operator` catalog,
  bounded-depth search. Wired to `build_fulfill_delivery_bt`.
- Expand — trade-run catalog (SellAll terminal) + first combined
  goal `fulfill_delivery_with_retrofit` that folds a shipyard
  detour into a delivery BT. Plain BT builders stay as fallbacks.
- Embrace — real fuel state threaded through via
  `fuel_fraction_to_bucket(ship.fuel.fraction())`. Passenger /
  courier catalog (`DeliverPassengers` operator, no cargo
  mechanics). Standalone retrofit catalog. Second combined goal
  `trade_run_with_retrofit`.
- Complete coverage — crew_mission (`Rest` operator + `rested`
  state, ordering enforced by the `delivered && rested` goal
  predicate) and rest_crew catalogs ported. All 7 goal types
  now planner-backed.
- Cost-weighted search — BFS swapped for Dijkstra. `Operator::cost`
  scores travel by fuel-bucket burn (everything else flat 1),
  so combined plans pick cheapest rather than shortest-by-steps.
- Retire fallbacks — all six `build_*_handauthored` siblings
  deleted (-216 lines). Each `build_*` wrapper calls its
  `try_*_planned` counterpart; on the (never-observed in prod)
  `None` result, returns `Idle` BT and replans next tick.

**Bundle 4 tranche:**
- S — **Structural-warning events.** `market_crisis` +
  `market_recovered` (threshold 200 shortage ticks with 50-tick
  hysteresis), `facility_starving` + `facility_recovered`
  (condition <0.3 / >0.5). Emitted from a `scan_crisis_events`
  pass after `tick_markets`, hysteresis-flagged on
  `MarketHistory.crisis_notified` + `FacilityOperations.starving_notified`.
- H — **Facility recipe profitability gate.** `recipes::execute`
  takes optional `prices` and skips batches where
  `output_value < input_value × MIN_RECIPE_MARGIN_RATIO` (0.5,
  so ≥50% loss → skip; normal fluctuation runs through).
- G — **Docking-fee licensing burn.** `commit_dock` burns
  `DOCK_FEE_BURN_PCT=0.30` of every dock fee to the void; 70%
  goes to the facility as before. Primary lever against long-run
  credit inflation.
- Q — **WorldTuning scaffold.** All ten tonight-commit tuning
  constants collected into `sim_core/src/simulation/tuning.rs`.
  Loaded at boot from `priv/data/tuning.json` (missing file →
  defaults; partial JSON → `serde(default)` fills the gaps).
  Inspectable via `GET /api/admin/tuning`. Call sites read
  `WorldTuning::current()`.
- K — **Population wealth as a real variable.**
  `PopEconomy.total_wealth` now responds to production + consumption
  at local market prices (rate 10% — enough to observe drift,
  small enough not to destabilise). Smoke observed producer pops
  (Veritas) gaining + consumer pops losing as expected.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kalcode/spaceprojectsim](https://github.com/Kalcode/spaceprojectsim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
