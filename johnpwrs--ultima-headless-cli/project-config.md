---
trigger: always_on
description: A headless Ultima Online client that speaks the full UO network protocol with no window, no rendering, and no audio. Designed to be controlled by an agent (human or AI) through a CLI or daemon interface.
---

# ClassicUO Headless Client — Agent Guide

A headless Ultima Online client that speaks the full UO network protocol with no window, no rendering, and no audio. Designed to be controlled by an agent (human or AI) through a CLI or daemon interface.

---

## Quick Start

### 1. Configure credentials

Edit `headless/.env`:

```env
UO_HOST=login.uorenaissance.com
UO_PORT=2593
UO_USER=youraccount
UO_PASS=yourpassword
UO_VERSION=5.0.8.3
UO_SERVER=0        # auto-select server index (comment out to pick manually)
UO_CHAR=0          # auto-select character index (comment out to pick manually)
```

### 2. Build

```bash
cd headless
dotnet build
```

### 3. Run — Interactive mode

```bash
dotnet run
```

You get a `cuo>` prompt. The client auto-connects and logs in using `.env`. Type `help` for all commands.

### 4. Run — Daemon mode (recommended for agents)

```bash
dotnet run -- --daemon &
```

- Commands go to: `echo "command" >> /tmp/cuocmd`
- All output goes to: `tail -f /tmp/cuolog`
- The session stays alive. Send commands at any time without restarting.

```bash
# Send a command
echo "status" >> /tmp/cuocmd

# Watch output live
tail -f /tmp/cuolog

# Read last N lines
tail -20 /tmp/cuolog
```

---

## Command Reference

### Login flow (manual, if UO_SERVER / UO_CHAR not set)

| Command | Description |
|---------|-------------|
| `connect <host> <port> <user> <pass>` | Connect to a server |
| `servers` | List available shards after connecting |
| `server <index>` | Select a shard by index |
| `chars` | List character slots |
| `char <index>` | Enter the world with that character |

---

### Character info

| Command | Aliases | Description |
|---------|---------|-------------|
| `status` | `stat` | Full stats: HP, mana, stamina, STR/DEX/INT, gold |
| `pos` | `position` | Current X, Y, Z, direction, map index |
| `world` | `info` | Login state, map, mobile/item counts |
| `skills` | `sk` | All skills with value and lock status (↑up / ↓dn / ■locked) |
| `inv` | `inventory`, `equip` | Equipped items (by layer) + backpack contents |

---

### Movement

| Command | Description |
|---------|-------------|
| `walk <dir>` | Walk one tile. Directions: `n s e w ne nw se sw` (or full names) |
| `walk <dir> run` | Run instead of walk |

**Examples:**
```
walk n
walk ne run
walk southwest
```

Position updates automatically after each confirmed step.

---

### Combat

| Command | Aliases | Description |
|---------|---------|-------------|
| `war` | `warmode` | Toggle war/peace mode |
| `attack <serial>` | `atk`, `a` | Attack a specific mobile. **Only works on grey/red targets.** Blue (Innocent) targets are refused. |
| `attacknearest` | `an` | Attack nearest grey/red mobile with clear **line of sight** |
| `nearest` | `nearestmob` | Show the closest mobile (name, serial, distance) |

**Important:** `attacknearest` (`an`) automatically:
1. Filters to CanBeAttacked / Criminal / Enemy / Murderer only
2. Checks line of sight via map data
3. Skips targets behind walls

---

### Interaction

| Command | Aliases | Description |
|---------|---------|-------------|
| `click <serial>` | | Single-click (request name/status bar) |
| `use <serial>` | `dclick` | Double-click (open container, use item, open door) |
| `wear <serial> <layer_hex>` | | Equip an item to a body layer (lift + wear) |
| `unequip <serial>` | `topack` | Move an equipped item into the backpack |

**Layer hex values:**
```
01 = One-handed weapon    02 = Two-handed weapon (bows go here)
03 = Shoes                04 = Pants / Legs
05 = Shirt / Chest        06 = Hat / Helm
07 = Gloves               08 = Ring
09 = Talisman             0A = Neck
0B = Ring (alt)           0C = Earrings
0D = Cloak                0E = Belt
0F = Tunic                10 = Robe
11 = Boots                12 = Skirt
13 = Robe (alt)           14 = Misc
15 = Backpack             16 = Mount
```

**Example — equip crossbow:**
```
use 401F10E1        # open backpack to load inventory
inv                 # find crossbow serial
unequip 401F10E4   # remove melee weapon from slot 1
wear 401F10E6 02   # equip crossbow to two-handed slot
```

---

### Speech

| Command | Aliases | Description |
|---------|---------|-------------|
| `say <text>` | `s` | Say in public chat |
| `yell <text>` | `y` | Yell (heard further away) |
| `emote <text>` | `em` | `*emote*` action |
| `whisper <text>` | `wh` | Whisper (private, close range) |

Server commands use the same `say` interface:
```
say [help
say [skills
say [young
say [where
```

---

### World / nearby entities

| Command | Aliases | Description |
|---------|---------|-------------|
| `mobiles [range]` | `mobs`, `m` | List nearby mobiles with name, serial, notoriety, HP, distance. Default range: 18 |
| `items [range]` | `i` | List nearby ground items with serial, graphic, amount |
| `nearest` | | Closest mobile |

**Notoriety meanings:**
- `Innocent` — blue, attacking makes you criminal
- `CanBeAttacked` — grey, safe to attack
- `Criminal` — grey, already flagged
- `Enemy` — orange, attackable
- `Murderer` — red, attackable
- `Invulnerable` — cannot be harmed

---

### Map & pathfinding


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnpwrs/ultima-headless-cli](https://github.com/johnpwrs/ultima-headless-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
