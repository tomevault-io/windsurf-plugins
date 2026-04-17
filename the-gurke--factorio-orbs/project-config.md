---
trigger: always_on
description: **Orbs** is a comprehensive magical modification for Factorio that introduces a complete magic system centered around orbs, runes, and alchemy. The mod fundamentally transforms the gameplay experience by replacing traditional electricity-based automation with magical alternatives, while maintaining the core factory-building mechanics.
---

# Orbs Mod Documentation

## Overview

**Orbs** is a comprehensive magical modification for Factorio that introduces a complete magic system centered around orbs, runes, and alchemy. The mod fundamentally transforms the gameplay experience by replacing traditional electricity-based automation with magical alternatives, while maintaining the core factory-building mechanics.

For humans: The following is written for our lovely robot coding friends and contains a lot of spoilers! You might not want to read this.

## Key Features

### 1. Magical Items System

**Primary Items:**
- **Magic Orb**: Base magical energy item with stack size of 5
- **Active Magic Shard**: Spoils in 30 seconds to inactive shards (stack size 200)
- **Inactive Magic Shard**: Spoiled shards that can be banished or recycled
- **Conjuration Orb**: Special orb for crafting conjuration machines
- **Haste Orb**: Speed module that only works in conjuration machines (+20% speed)
- **Cleansing Orb**: Pollution reduction module for conjuration machines (-30% pollution)
- **Productivity Orb**: Productivity module for conjuration machines (+15% productivity)

**Flux Orbs:**
- **Alpha State**: Spoils to Beta in 30 seconds
- **Beta State**: Spoils to Gamma in 1 second  
- **Gamma State**: Spoils to Alpha in 1 second

**Volatile Orbs:**
- Created as a byproduct of essence of stability
- Five variants (Q, R, S, T, U) that explode after 40 seconds
- Need to be neutralized through operations that mimick the simple group of order 6.

### 2. Magical Machines

**Conjuration Machine:**
- Purple-tinted assembler that only crafts "orbs" category recipes
- Uses steam power and emits pollution
- 3 module slots accepting haste orbs, cleansing orbs, and productivity orbs
- Can benefit from resonance spire beacons

**Rune Transformer:**
- Transforms rune words in cycling patterns
- Based on stone furnace, no power required
- Recipes change dynamically during gameplay

**Rune Altar:**
- Creates rune research packs from specific rune sequences
- Requires exact placement: Vitae, Ignis, Tempus, Terra, Umbra, Mortis

**Resonance Spire:**
- Beacon that provides productivity effects to nearby conjuration machines
- Unlocked through rune research

### 3. Research System

**Research Packs:**
- **Contraption Research Pack**: Replaces automation science pack
- **Magical Research Pack**: Early game science pack to quickly unlock quality of life techs
- **Conjuration Research Pack**: Made from active magic shards + copper cables
- **Divination Research Pack**: Made from divination essence + serendipity + luck
- **Rune Research Pack**: Created from specific

Note: this mod uses "research" instead of "science" since we don't do science.

### 4. Recipe Categories

**Conjuration Recipes** (orbs category):

- Conjure Shards: 1-32 orb variants creating n² shards
- Replicate Shards: 50% chance bonus shards
- Manifest Orb: 10 active shards → 1 magic orb
- Transfigure: Create specialized orbs
- Neutralize: Convert specialized orbs back to magic orbs

**Early-game Fuel Recipes:**

- Light Wood: Wood → Burning Wood (4MJ fuel, decays in 2 min)
- Light Coal: Coal → Burning Coal (8MJ fuel, decays in 5 min)
- Fire through Friction: Stick + Wood → 10% chance Burning Wood

### 5. Integration with Nanobots2

**Thematic Overrides:**

- Nano gun → Magic Wand
- Nano ammo → Summoning Essences
- Technologies rebranded with magical themes
- Magical ingredients replace technological components

### 6. Game Balance Changes

**Fuel System:**

- Base wood/coal have no fuel value
- Must be lit on fire to become usable fuel
- Burning fuels decay over time to glimming remains

**Inserter System:**

- Burner inserters work at fast inserter speed
- Magic inserters require no power
- All inserters except magic ones require fuel

**Transportation:**

- Red belts require lqiuid stability to craft
- Gold plates provide speed boost flooring

## File Structure

### Core Data Files

- **`data.lua`**: Main data loader, includes all other data files
- **`categories.lua`**: Item groups, subgroups, and recipe categories
- **`item.lua`**: All item definitions including orbs, shards, and tools
- **`recipe.lua`**: All recipe definitions and generation logic
- **`entities.lua`**: Machine and entity definitions
- **`technology.lua`**: Research tree and technology definitions

### Runtime Control

- **`control.lua`**: Runtime scripts for telekinesis, soul collection, orbs research rewards, and rune transformation system
- **`rune-chains.lua`**: Defines cyclic transformation patterns for rune words

### Modifications

- **`data-final-fixes.lua`**: Late-stage modifications to base game and other mods
- **`removals.lua`**: Removes unwanted base game content and handles dependencies

### Localization

- **`locale/en/strings.cfg`**: All text strings for items, recipes, and technologies

### Graphics

- **`graphics/`**: custom graphics for all magical items and effects

## Magic System Mechanics

### Orb Conjuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/the-gurke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
