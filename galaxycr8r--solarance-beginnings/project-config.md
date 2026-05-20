---
trigger: always_on
description: > Read this first. Everything else in this file is mechanics. This is philosophy.
---

# Software Design Principles

> Read this first. Everything else in this file is mechanics. This is philosophy.

---

## The Mental Model

The server is built around one invariant: **state lives in tables, transitions live in reducers.**

Every bug in production manifests as "the wrong thing is in a table" or "the wrong reducer ran." That's it. If you keep that model sharp, you can diagnose anything from `spacetime logs` alone.

There are three layers and they must stay separate:

```
tables/     — What exists. Pure schema. No logic.
logic/      — What happens. Reducers + helper functions.
definitions/ — What starts as true. Seed data for init.
```

Don't let logic leak into table definitions. Don't let schema decisions be made inside reducers. When you find yourself writing a method on a table struct that calls back into the DSL — stop and ask whether that belongs in `logic/` instead.

---

## Deep Modules

This codebase is organised around **deep modules** (Ousterhout, *A Philosophy of Software Design*): each module should have a **simple interface and a complex implementation**. The interface is the public contract. The implementation is the mess you're paid to hide.

A shallow module has an interface almost as complex as its body — it adds a layer without adding abstraction. A deep module genuinely simplifies the caller's world.

**Measure depth by this ratio: how much does the caller need to know vs. how much is hidden?**

The `spacetimedsl` crate is a perfect example of a deep module. A caller writes `dsl.get_player_by_id(&id)?` and has no idea that SpacetimeDB index access, row deserialization, and error normalisation just happened. That complexity is paid once, hidden forever.

Write your logic functions the same way:

```
// Shallow — caller must orchestrate the whole operation
pub fn get_ship_type(dsl, ship_id) -> ShipTypeDefinition
pub fn get_ship_status(dsl, ship_id) -> ShipStatus
pub fn compute_new_velocity(status, type, controller) -> Vec2
pub fn save_velocity(dsl, sobj_id, velocity)

// Deep — caller states intent, not steps
pub fn apply_movement_tick(dsl, controller) -> Result<(), String>
```

If a caller has to call three of your functions in the right order to do one logical thing, your module is shallow. Pull that sequencing downward.

---

## The Debugging Contract

In production you have exactly one tool: `spacetime logs`. Design everything to make those logs useful.

**Every reducer is an audit trail entry.** Each state change has exactly one reducer that caused it. When something goes wrong in the live game, you will grep the logs for a reducer name and a row ID. That grep must find the answer.

This means:

1. **Reducers must be narrow.** One reducer = one conceptual operation. If a reducer does three unrelated things, a log line doesn't tell you which thing failed.

2. **Errors must carry context.** `"Ship not found"` is useless. `"Ship not found (likely docked), removing update timer for player ID: {id}"` is a diagnosis. Every `Err(...)` string should answer: *what* failed, *where*, and *why the code took that branch*.

3. **Timer reducers are the heartbeat.** `timer_update_all_ship_movement_controllers`, `station_production_schedule_reducer`, and their kin fire constantly. When something drifts, it drifted on a tick. Log the station ID, the sector, the ship — not just "tick completed".

4. **Helper functions propagate errors, they don't swallow them.** Use `?` and let the reducer surface the failure. A helper that returns `Ok(())` after silently catching an error breaks the audit trail.

---

## Information Hiding

Every decision you make inside a module that a caller doesn't need to know about is good engineering. Every detail that leaks across a module boundary is future maintenance debt.

**Concretely:**

- The `stations/production.rs` dispatcher (`match blueprint.get_category() { ... }`) is internal to the production tick. Callers don't know it exists. They call `process_station_production_tick`. Good.
- The `CreateShipMovementController` struct is an implementation detail of ship creation. Callers in `logic/ships/creation.rs` construct it, but nothing outside `logic/ships/` should need to know its fields. If it does, you've leaked a detail.
- `get_player_ship_and_sobj` in `players.rs` hides a two-step lookup (controller → sobj → ship). Any code that re-implements those two steps inline is a missed opportunity for this function.

Before adding a parameter to a function, ask: *is there a way to hide this inside the function instead?* Before making a type public, ask: *who outside this module actually needs to name this type?*

---

## The Three Rules for New Code

**1. Make the interface smaller than the implementation.**
If your public function count grows at the same rate as your line count, you're writing shallow modules. Group related operations and hide the sequencing.

**2. Errors are documentation.**
Every `Err(string)` will eventually be the only clue you have when the game is live and a player files a bug report at 2am. Write it for that person.

**3. The DSL is the database. Don't re-abstract it.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GalaxyCr8r/solarance-beginnings](https://github.com/GalaxyCr8r/solarance-beginnings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
