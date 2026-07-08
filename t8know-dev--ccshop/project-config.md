---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **CC:Tweaked (ComputerCraft Tweaked)** Lua application for modded Minecraft. It implements an interactive item display shop using physical pedestals and a monitor UI. The shop supports multiple categories, materials, quantity selection, and payment via Numismatics depositor, integrated with Applied Energistics 2 (AE2) for stock checking.

## Running the Code

There is no build system. Code runs directly in-game on a ComputerCraft computer:
```
lua shop.lua
```

The program runs until terminated with Ctrl+T (in-game). Ensure all required peripherals are attached and configured in `config.lua`.

**Directory layout:** The script expects `config.lua`, `items.lua`, and `db.lua` to be in the `/ccshop/` directory (same folder as `shop.lua`). Logs are written to `/ccshop/shop_debug.log` and purchases to `/ccshop/purchases.json`.

## Architecture

The system is split across multiple Lua files for modularity:

- **shop.lua** — Main orchestrator that loads modules and runs the main loops.
- **config.lua** — Peripheral names, messages, timeouts, pedestal list, and validation function.
- **items.lua** — Categories, materials, quantity tiers, price definitions, and helper functions.
- **db.lua** — Purchase logging (ndjson format) to `/ccshop/purchases.json`.
- **modules/** — Modular components with single responsibilities:
  - **logging.lua** — Logging utilities with log level control.
  - **config.lua** — Enhanced configuration loading and validation.
  - **peripherals.lua** — Peripheral management, AE2 cache, relay helpers.
  - **state.lua** — Centralized state management with observer pattern and reset functions.
  - **pedestal.lua** — Pedestal rendering and management.
  - **ui.lua** — Basalt UI creation and updates.
  - **screens.lua** — Screen rendering logic.
  - **events.lua** — Event handling and state transitions.
  - **payment.lua** — Payment detection and idle timeout monitoring.
  - **dispense.lua** — Item dispensing from ME interfaces to barrel. Runs as 4th parallel coroutine.

The modules use dependency injection to avoid circular dependencies and enable testing.

### Flow of Screens

The shop operates as a five‑screen state machine with Screen 3 having two sub‑states:

1. **Category selection** — Welcome screen. Pedestals show category icons (iron, certus quartz, redstone, dye). Right‑click selects a category. No cancel button. Idle timeout does not apply.

2. **Material selection** — Shows only materials belonging to the chosen category that have sufficient stock in AE2. Right‑click selects a material, left‑click returns to category selection. Cancel button visible.

3. **Quantity selection and payment** — Two sub‑states:
   - **3A: Selecting a quantity** — Pedestals show available quantity tiers (from `minQty` up to AE2 stock). Right‑click chooses a quantity, left‑click returns to material selection. Cancel button visible.
   - **3B: Awaiting payment** — Triggered immediately after quantity selection. Selected pedestal label changes to `"[<qty>]"`. Depositor is configured and unlocked. Monitor shows calculated price and insert instruction. Cancel button visible.

4. **Dispensing progress** — Progress bar and counter as items move from ME interfaces to barrel in batches of 64. No cancel button. DispenseMonitorLoop handles progression.

5. **Thank‑you screen** — Shows "Purchase complete. Thank you!" and auto‑returns to screen 1 after `CONFIRM_DELAY` (3s). No cancel button.

### Peripheral Integration

- **Display pedestals** (Pedestals mod) — Show items/labels, receive `pedestal_left_click`/`pedestal_right_click` events.
- **AE2 adapter** (`ae2cc_adapter`) — Queries available stock for material items.
- **Numismatics depositor** — Accepts spur currency, signals payment via redstone relay.
- **Speaker** (`speaker_212`) — Plays harp sound for selection confirm (right‑click) and bass sound for cancel/back actions (left‑click, cancel button).
- **Monitor** — Shows Basalt UI with header, hint line, and cancel button.
- **Barrel** (`minecraft:barrel_59`) — Receives dispensed items; Item Storage API peripheral.
- **ME Interfaces** (list in config) — Stock items configured by player, drained via `pushItem()`.

### Concurrency

The main orchestrator runs four coroutines in parallel using `parallel.waitForAny`:
- `basalt.run()` — Basalt UI main loop.
- `events.eventLoop` — Listens for pedestal click events and handles state transitions.
- `payment.paymentMonitorLoop` — Monitors payment detection and idle timeouts.
- `dispense.dispenseMonitorLoop` — Handles item dispensing from ME interfaces to barrel.

## Key Data Structures

### Pedestal List (`config.lua`)
```lua
PEDESTALS = {
  "display_pedestal_12",  -- leftmost
  "display_pedestal_10",
  "display_pedestal_11",
  "display_pedestal_5",
  "display_pedestal_9",
  "display_pedestal_7",   -- rightmost
}
```
The script centers the currently active options across all pedestals; unused pedestals are cleared.

### Categories (`items.lua`)
```lua
CATEGORIES = {
  { label = "Metals",   item = "minecraft:iron_ingot" },
  { label = "Crystals", item = "ae2:certus_quartz_crystal" },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [t8know-dev/ccshop](https://github.com/t8know-dev/ccshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
