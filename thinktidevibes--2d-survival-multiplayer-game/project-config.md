---
trigger: always_on
description: Guide outlining the steps to add new consumable resources or gatherable nodes.
---

# Guide: Adding New Resources and Nodes

This guide outlines the steps required to add new consumable resources (like Pumpkins) or gatherable nodes (like Metal Ore) to the 2D Survival Multiplayer game. Follow these steps to ensure consistency with the existing architecture.

## General Workflow

1.  **Define Data (Server):** Add necessary item definitions and entity table structures.
2.  **Implement Logic (Server):** Create server-side reducers for interaction, harvesting, spawning, and respawning.
3.  **Integrate (Server):** Update server logic (seeding, respawning) to include the new resource.
4.  **Generate Bindings (CLI):** Run `spacetime generate` **before** making client-side changes that depend on new server types or reducers.
5.  **Add Assets (Client):** Place images for the resource doodad and its corresponding item.
6.  **Client-Side State Management:**
    *   Update `useSpacetimeTables.ts` to manage the new entity's state.
    *   Update `App.tsx` to fetch and pass the new entity's data.
    *   Update `GameScreen.tsx` to receive and pass down the new entity's data.
7.  **Client Rendering & Logic (Client):**
    *   Create rendering utilities (e.g., `pumpkinRenderingUtils.ts`) including image preloading.
    *   Update type guards (`typeGuards.ts`).
    *   Update entity filtering (`useEntityFiltering.ts`).
    *   Integrate into the main rendering loop (`GameCanvas.tsx`).
    *   Update interaction finding (`useInteractionFinder.ts`).
    *   Update interaction labels (`labelRenderingUtils.ts`).
    *   Update input handling (`useInputHandler.ts`) to call server reducers.
    *   Update item icon mapping (`itemIconUtils.ts`).
8.  **Testing:** Test spawning, interaction/harvesting, item yield, respawning, and UI rendering thoroughly.

---

## Adding a Consumable Resource (e.g., Pumpkin)

This resource type is picked up directly by the player, disappears, and respawns after a timer. It yields an item (e.g., "Pumpkin" item).

### Server (`server/`)

1.  **Define Item:**
    *   In `src/items_database.rs`, within the `get_item_definitions()` function's returned vector, add an `ItemDefinition` for the yielded item (e.g., "Pumpkin").
        ```rust
        ItemDefinition {
            id: 0, // Will be auto-assigned
            name: "Pumpkin".to_string(),
            description: "A ripe pumpkin, good for eating or crafting.".to_string(),
            category: ItemCategory::Consumable, // Or ItemCategory::Material if not directly edible
            icon_asset_name: "pumpkin.png".to_string(), // Matches asset in client/src/assets/items/
            is_stackable: true,
            stack_size: 10,
            // ... other fields as necessary
            damage: None,
            is_equippable: false,
            equipment_slot_type: None,
            fuel_burn_duration_secs: None,
        }
        ```
2.  **Create Resource Module:**
    *   Create a new file: `src/pumpkin.rs`.
3.  **Define Entity Struct (`src/pumpkin.rs`):**
    *   Define the `Pumpkin` struct:
        ```rust
        use spacetimedb::{table, ReducerContext, Identity, Timestamp, log};
        use crate::collectible_resources::{validate_player_resource_interaction, collect_resource_and_schedule_respawn, BASE_RESOURCE_RADIUS, PLAYER_RESOURCE_INTERACTION_DISTANCE_SQUARED};
        use crate::TILE_SIZE_PX; // If needed for positioning logic, though not directly for basic consumable

        #[table(name = pumpkin, public)]
        #[derive(Clone, Debug)]
        pub struct Pumpkin {
            #[primary_key]
            #[auto_inc]
            pub id: u64,
            pub pos_x: f32,
            pub pos_y: f32,
            pub chunk_index: u32,
            pub respawn_at: Option<Timestamp>,
        }

        // Constants
        pub const PUMPKIN_YIELD_ITEM_NAME: &str = "Pumpkin"; // Name of the item defined in items_database.rs
        pub const PUMPKIN_YIELD_AMOUNT: u32 = 1;
        pub const PUMPKIN_RESPAWN_TIME_SECS: u64 = 180; // Example: 3 minutes
        pub const PUMPKIN_RADIUS: f32 = BASE_RESOURCE_RADIUS; // Or a custom radius

        // Spawning density and minimum distance constants (adjust as needed)
        pub const PUMPKIN_DENSITY_PERCENT: f32 = 0.5;
        pub const MIN_PUMPKIN_DISTANCE_SQ: f32 = (PUMPKIN_RADIUS * 2.0 + 50.0) * (PUMPKIN_RADIUS * 2.0 + 50.0);
        pub const MIN_PUMPKIN_TREE_DISTANCE_SQ: f32 = (PUMPKIN_RADIUS + crate::tree::TREE_RADIUS + 50.0) * (PUMPKIN_RADIUS + crate::tree::TREE_RADIUS + 50.0);
        pub const MIN_PUMPKIN_STONE_DISTANCE_SQ: f32 = (PUMPKIN_RADIUS + crate::stone::STONE_RADIUS + 50.0) * (PUMPKIN_RADIUS + crate::stone::STONE_RADIUS + 50.0);
        // Add similar constants for other resources if needed (e.g., MIN_PUMPKIN_CORN_DISTANCE_SQ)
        ```
4.  **Implement Interaction Reducer (`src/pumpkin.rs`):**
    *   Create a reducer `interact_with_pumpkin(ctx: &ReducerContext, pumpkin_id: u64)`.
        ```rust
        #[spacetimedb::reducer]
        pub fn interact_with_pumpkin(ctx: &ReducerContext, pumpkin_id: u64) -> Result<(), String> {
            let sender_id = ctx.sender;

            let pumpkin_entity = ctx.db.pumpkin().id().find(pumpkin_id)
                .ok_or_else(|| format!("Pumpkin with ID {} not found.", pumpkin_id))?;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thinktidevibes/2D-Survival-Multiplayer-Game](https://github.com/thinktidevibes/2D-Survival-Multiplayer-Game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
