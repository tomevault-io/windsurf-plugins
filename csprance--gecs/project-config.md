---
trigger: always_on
description: transforms[i].transform.global_position += velocities[i].velocity * delta
---

### GECS AI Coding Guide

Concise, codebase-specific instructions for AI agents. Focus only on proven patterns in this repo.

#### Core Runtime (under `addons/gecs/ecs/`)

Entity (`entity.gd`): Node holding components (data) + relationships. Provides `add_component()`, `has_component()`, `add_relationship()`.
Component (`component.gd`): Resource, data-only `@export` fields. Emits `property_changed` manually to trigger observers.
System (`system.gd`): Override `query()` returning a `QueryBuilder`; implement `process(entities, components, delta)`. Use `iterate([...])` in query for batch column access (components array order matches iterate list). Optional `sub_systems()` returns `[QueryBuilder, Callable]` or `[QueryBuilder, Callable, SystemTimer]` tuples — the optional third element gates the subsystem to only run when the timer ticks.
Observer (`observer.gd`): Reactive node: override `query() -> QueryBuilder` and chain event modifiers (`.on_added()`, `.on_removed()`, `.on_changed([&"prop"])`, `.on_match()`, `.on_unmatch()`, `.on_relationship_added([...])`, `.on_relationship_removed()`, `.on_event(&"name")`). Implement `each(event, entity, payload)` as the single dispatch callback. Use `sub_observers()` to compose multiple reactive axes in one node. `CHANGED` events require the component to emit `property_changed` manually in a setter.
World (`world.gd`): Owns entities, systems, observers, archetype & relationship indices. Provides `world.query` (pooled `QueryBuilder`), archetype cache, enabled/disabled filtering baked into signatures.
ECS (`ecs.gd`): Autoload singleton exposing `ECS.world` and `ECS.process(delta, group?)`.

#### QueryBuilder Essentials (`query_builder.gd`)

Chaining: `with_all([...])`, `with_any([...])`, `with_none([...])`, `with_relationship([...])`, `without_relationship([...])`, `with_group([...])`, `without_group([...])`, `.enabled()`, `.disabled()`, `.iterate([CompA, CompB])`.
Component property filters supported via dictionaries: `with_all([{C_Health: {"current": {"_lt": 20}}}])`.
`execute()` returns entities; `archetypes()` returns matching archetypes for high-performance column access.
Cache keys (FNV-1a) reused between World `_query` and archetype retrieval; relationship changes invalidate query cache.

#### Archetype & Performance Model

Entities grouped by component signature (+ enabled bit) → O(1) query intersection using archetype match + result flattening only when needed. Enable/disable moves entity to distinct archetype; `.enabled()` / `.disabled()` skip entity-level filtering.
Use `iterate()` or `archetypes()` inside systems for tight loops: access columns via `archetype.get_column(ComponentScript.get_instance_id())`.
Parallel processing: set `parallel_processing=true` and `parallel_threshold` on a System; only use pure data logic (no scene tree access) inside `process()` when parallel.

#### Relationships (`relationship.gd`)

Create: `Relationship.new(C_Likes.new(), target_entity)` or with property queries: `Relationship.new({C_Buff: {'duration': {'_gt':10}}}, {C_Player: {'level': {'_gte':5}}})`.
Wildcard: pass `null` as relation or target. Removal supports count limiting: `entity.remove_relationship(Relationship.new(C_Damage.new(), null), 2)`.
Reverse queries: `with_reverse_relationship([...])` maps target → sources via index.

#### CommandBuffer (`command_buffer.gd`)

Callable-based deferred execution for safe structural changes during system iteration. Each queue method appends a lambda with baked-in `is_instance_valid` guard to `Array[Callable]`. Commands execute in exact queued order.

System property: `cmd: CommandBuffer` (lazy-initialized). Queue methods: `add_component()`, `remove_component()`, `add_components()`, `remove_components()`, `add_entity()`, `remove_entity()`, `add_relationship()`, `remove_relationship()`, `add_custom()`. Inspection: `is_empty()`, `size()`, `get_stats()`. Manual: `execute()`, `clear()`.

Flush modes (`command_buffer_flush_mode: FlushMode` export on System):
- **FlushMode.PER_SYSTEM** (default): auto-executes after each system completes
- **FlushMode.PER_GROUP**: auto-executes after all systems in group complete
- **FlushMode.MANUAL**: requires explicit `ECS.world.flush_command_buffers()`

Pattern: use `cmd.remove_entity(entity)` instead of `ECS.world.remove_entity(entity)` inside system `process()` for safe forward iteration. Use `cmd.add_component(entity, comp)` instead of `entity.add_component(comp)` when modifying entities during iteration.

#### SystemTimer (`system_timer.gd`)

Tick rate control for systems. Systems run every frame by default; assign a `SystemTimer` to `tick_source` to throttle execution. `set_tick_rate(interval, single_shot?)` is a convenience that creates and assigns the timer, returning it for sharing.

Shared timers: multiple systems referencing the same `SystemTimer` instance are guaranteed to tick on the same frame. Timers are advanced once per group in `World.process()` before systems run. Overshoot is carried forward to prevent drift.

```gdscript
# Throttle a system
func setup(): set_tick_rate(0.5)  # every 500ms

# Share a timer
var timer = system_a.set_tick_rate(0.2)
system_b.tick_source = timer

# One-shot (fire once after delay)
func setup(): set_tick_rate(3.0, true)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [csprance/gecs](https://github.com/csprance/gecs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
