---
trigger: always_on
description: Guide for adding new deployable items (e.g., structures, placeables) to the world.
---

# Guide: Adding New Deployable Items

This guide outlines the general steps to add a new deployable item (like campfires, storage boxes, or sleeping bags) to the game. Replace bracketed placeholders (e.g., `[Item Name]`, `[item_name]`) with your specific item's details.

## 1. Server: Define the Item

*   **File:** [`server/src/items_database.rs`](mdc:server/src/items_database.rs)
*   **Action:** Add a new `ItemDefinition` struct entry for your `[Item Name]`.
        *   Set `category: ItemCategory::Placeable`.
    *   Set `icon_asset_name: "[item_name].png"`.
    *   Configure `is_stackable` (usually `false` for placeables), `stack_size` (usually `1`), `is_equippable: false`.
    *   *(Optional)* Define a crafting recipe later if needed.

## 2. Server: Create the Entity Logic

*   **New File:** `server/src/[item_name].rs`
*   **Action:**
    *   Define the `[ItemName]` struct (e.g., `SleepingBag`) with `#[spacetimedb::table(name = [item_name], public)]`.
    *   **Required Fields:** `id: u32` (PK, auto_inc), `pos_x: f32`, `pos_y: f32`, `chunk_index: u32`, `placed_by: Identity`, `placed_at: Timestamp`. Add other fields as needed for the item's function.
    *   Add collision/interaction constants (copy/adapt from [`server/src/wooden_storage_box.rs`](mdc:server/src/wooden_storage_box.rs) or [`server/src/campfire.rs`](mdc:server/src/campfire.rs)).
    *   Implement `place_[item_name]` reducer:
        *   Find your item's definition ID.
        *   Validate the item instance (ownership, type).
        *   Validate placement distance/collision.
        *   Consume the item from player inventory/hotbar (using helpers from [`server/src/items.rs`](mdc:server/src/items.rs) or [`server/src/inventory_management.rs`](mdc:server/src/inventory_management.rs)).
        *   Calculate `chunk_index` using `calculate_chunk_index` from [`server/src/environment.rs`](mdc:server/src/environment.rs).
        *   Insert the new `[ItemName]` entity.
    *   Implement `interact_with_[item_name]` reducer (at least basic distance check if interaction is needed).
    *   Implement `pickup_[item_name]` reducer (if applicable):
        *   Validate interaction.
        *   Check if pickup conditions met (e.g., empty).
        *   Add item back to player inventory using `add_item_to_player_inventory` from [`server/src/items.rs`](mdc:server/src/items.rs).
        *   Delete the entity if item added successfully.
    *   Implement `validate_[item_name]_interaction` helper (if needed).
*   **File:** [`server/src/lib.rs`](mdc:server/src/lib.rs)
*   **Action:**
    *   Add `mod [item_name];`
    *   Add `pub use crate::[item_name]::[ItemName];` near the end of the file.
    *   Add `use crate::[item_name]::[item_name] as [ItemName]TableTrait;` to the table trait imports near the top.

## 3. Server: Update Starting Items (Optional)

*   **File:** [`server/src/starting_items.rs`](mdc:server/src/starting_items.rs)
*   **Action:** If players should start with this item, add an entry for `"[Item Name]"` to the `starting_inv_items` array.

## 4. Client: Map the Icon

*   **File:** [`client/src/utils/itemIconUtils.ts`](mdc:client/src/utils/itemIconUtils.ts)
*   **Action:**
    *   Import the icon: `import [itemName]Icon from '../assets/items/[item_name].png';`
    *   Add the mapping to `itemIcons`: `'[item_name].png': [itemName]Icon`.

## 5. Client: Create Rendering & Type Logic

*   **File:** [`client/src/config/gameConfig.ts`](mdc:client/src/config/gameConfig.ts)
*   **Action:** Define constants for the item's dimensions (e.g., `[ITEM_NAME]_WIDTH`, `[ITEM_NAME]_HEIGHT`).
*   **File:** [`client/src/utils/typeGuards.ts`](mdc:client/src/utils/typeGuards.ts)
*   **Action:**
    *   Import `[ItemName] as SpacetimeDB[ItemName]` from `../generated`.
    *   Add `is[ItemName](mdc:entity: any): entity is SpacetimeDB[ItemName]` type guard.
*   **New File:** `client/src/utils/[itemName]RenderingUtils.ts`
*   **Action:**
    *   Implement `render[ItemName](mdc:...)` to draw the item (using dimensions from `gameConfig`).
    *   Implement `preload[ItemName]Image(...)` if needed.

## 6. Client: Integrate Rendering & Placement

*   **File:** [`client/src/utils/renderingUtils.ts`](mdc:client/src/utils/renderingUtils.ts)
*   **Action:**
    *   Import `render[ItemName]` and `is[ItemName]`.
    *   Add `SpacetimeDB[ItemName]` to the `Entity` type union.
    *   In `renderGroundEntities` or `renderYSortedEntities` (depending on item type), add check `if (is[ItemName](mdc:entity))` and call `render[ItemName]`. *Crucial for drawing!* 
*   **File:** [`client/src/utils/placementRenderingUtils.ts`](mdc:client/src/utils/placementRenderingUtils.ts)
*   **Action:** Update `renderPlacementPreview`:
    *   Import dimensions from `gameConfig`.
    *   Add an `else if` check for `placementInfo.iconAssetName === '[item_name].png'` to set the correct `drawWidth` and `drawHeight`. *Crucial for placement preview size!* 
*   **File:** [`client/src/hooks/usePlacementManager.ts`](mdc:client/src/hooks/usePlacementManager.ts)
*   **Action:** In the `attemptPlacement` function's `switch` statement, add a `case '[Item Name]':` that calls the correct server reducer (`connection.reducers.place_[item_name](mdc:...)`). *Crucial for initiating placement!* 


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thinktidevibes/2D-Survival-Multiplayer-Game](https://github.com/thinktidevibes/2D-Survival-Multiplayer-Game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
