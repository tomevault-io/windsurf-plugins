---
trigger: always_on
description: Hyperion is a flecs-ECS Minecraft game engine (Rust, edition 2024). This file
---

# CLAUDE.md

Hyperion is a flecs-ECS Minecraft game engine (Rust, edition 2024). This file
records the repo-wide conventions an agent or contributor must follow. Skills
under the harness cover general house style; the rules here are hyperion's own.

## flecs modules: separate component registration from behavior

**The rule.** A flecs module is one of two kinds, never both:

- A **registration module** ONLY registers components. It calls
  `world.component::<T>()` (with traits like `add_trait::<flecs::Singleton>()`,
  `.meta()`, `.member(...)`), declares relationships, and builds prefabs. It
  installs no systems, no observers, and no behavior. Installing a singleton's
  default value with `world.set::<T>(..)` belongs here too, right after `T` is
  registered, because a `set` of an unregistered component is the same class of
  bug as a `get`.
- A **behavior module** holds the systems and observers. It runs no
  `world.component::<T>()` of its own. Instead, for every component `T` it
  touches, it `world.import::<TRegistrationModule>()` the registration module
  that owns `T`. flecs runs an imported module exactly once and before the
  importer's body, so every component a system reads is registered before the
  system is declared.

**Why this exists (ENG-11000).** The workspace builds flecs with
`flecs_manual_registration` (`Cargo.toml`, `[workspace.dependencies.flecs_ecs]`),
so a component must be registered before first use. The guard that enforces
this is a flecs `ecs_assert`, and flecs asserts are compiled out of release
builds. So the two profiles disagree:

- **Debug / dev profile** (`nix run .#smash`, `cargo test`): first use of an
  unregistered component aborts with
  `ECS_INVALID_OPERATION: Component <T> is not registered with the world before usage`.
- **Release profile** (every e2e gate binary): the assert is gone, the C side
  registers the component lazily on first use, and the code reads fine.

ENG-11000 was exactly this gap. `WorldTimeModule` did
`world.set(WorldTime::default())` without first registering `WorldTime` as a
`Singleton`. Every release gate stayed green (including a new `world-time-e2e`),
while `nix run .#smash` crash-looped on boot in the dev profile. A
release-binary gate structurally cannot catch a dev-only debug assert, so the
structure of the code, not the test suite, is what has to make this impossible.

**How the convention makes the class impossible.** If a system uses `T`, its
behavior module imports `T`'s registration module, and flecs's module-import
DAG runs that registration before the system is declared. There is no ordering
a contributor can pick that registers `T` after a use of `T`, because "uses `T`"
and "imports the module that registers `T`" are the same edit. Use-before-
register stops being a thing you can forget and becomes a thing the import graph
forbids.

**Second benefit: composability.** Because registration carries no behavior, a
consumer can import the *components* (the types) without importing the
*behavior* (the systems). The smash mock, for example, can import a
component-registration module plus one chosen physics behavior module without
dragging in the whole host or egress layer. Splitting registration out is what
makes "give me the types but not the systems" expressible.

### Skeleton

```rust
use flecs_ecs::prelude::*;

// ---- the components this domain owns ----
#[derive(Component, Debug, Clone, Copy)]
pub struct Health(pub f32);

#[derive(Component, Debug, Default)]
pub struct DamageTuning { pub multiplier: f32 }

/// Registration module: types and traits only, no systems or observers.
/// Behavior modules import this; it imports nothing behavioral.
#[derive(Component)]
pub struct CombatComponentsModule;

impl Module for CombatComponentsModule {
    fn module(world: &World) {
        world.component::<Health>();
        // A singleton: register with the trait, then install its default in
        // the same place. `add_trait::<flecs::Singleton>()` is load-bearing --
        // a bare `world.set` stores the value but never registers the type, and
        // that is the ENG-11000 abort in a dev build.
        world
            .component::<DamageTuning>()
            .add_trait::<flecs::Singleton>();
        world.set(DamageTuning::default());
    }
}

/// Behavior module: systems and observers. It registers no components; it
/// imports the registration module for every component it reads or writes.
#[derive(Component)]
pub struct CombatModule;

impl Module for CombatModule {
    fn module(world: &World) {
        world.import::<CombatComponentsModule>();

        system!("apply_damage", world, &mut Health, &DamageTuning($))
            .kind(id::<flecs::pipeline::OnUpdate>())
            .each(|(health, tuning)| {
                health.0 -= tuning.multiplier;
            });
    }
}
```

### The import-DAG ordering rule

- **Behavior imports registration.** Every behavior module begins by importing
  the registration module(s) for the components its systems and observers
  touch. Import the ones you use even if you believe a parent already imported
  them; flecs dedupes imports, so an extra edge is free and a missing one is a
  dev-profile crash.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperion-mc/hyperion](https://github.com/hyperion-mc/hyperion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
