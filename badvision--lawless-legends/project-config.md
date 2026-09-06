---
trigger: always_on
description: This document captures architectural knowledge for AI agents working in this part of the codebase. The OutlawEditor (Java) is agnostic to these details; this is where it matters.
---

# PackPartitions — Agent Context

This document captures architectural knowledge for AI agents working in this part of the codebase. The OutlawEditor (Java) is agnostic to these details; this is where it matters.

---

## What This Tool Does

`A2PackPartitions.groovy` is the build-time packer that transforms `world.xml` (the OutlawEditor game data file) into PLASMA source files, which are then compiled and assembled into Apple II binary modules loaded at runtime.

Pipeline:
```
world.xml  →  A2PackPartitions.groovy  →  .pla (PLASMA source)
           →  PLASMA compiler           →  .a (6502 assembly)
           →  ACME assembler            →  .b (binary module)
           →  disk image packer         →  disk image
```

---

## Game Flags

### Definition
Flags are defined in `world.xml` under:
```xml
<global>
  <sheets>
    <sheet name="Flags">
      <columns>
        <column name="name"/>
        <column name="number"/>
        <column name="description"/>
      </columns>
      <rows>
        <row number="95" name="KILL" description="?"/>
        ...
      </rows>
    </sheet>
  </sheets>
</global>
```

- 189 flags currently, numbered 0–188 (gaps allowed; up to 255 supported)
- **Flag numbers are manually assigned** in the sheet — the packer never auto-assigns them
- `description` is ignored by the packer (documentation only)

### Packer Behavior
`numberGameFlags()` scans every `interaction_get_flag`, `interaction_set_flag`, and `interaction_clr_flag` block across all scripts, collects the names, then cross-validates against the Flags sheet. Rules:
- Every flag name used in a script **must** exist in the sheet → hard assertion failure if missing
- Flags in the sheet but not used in any script → warning only
- Flag names are **case-insensitive** at pack time (`"Kill"`, `"KILL"`, `"kill"` all resolve to the same flag)

`humanNameToSymbol(name, true)` converts the name to a PLASMA constant:
- `"kill"` → `GF_KILL`
- `"player dead"` → `GF_PLAYER_DEAD`
- Non-alphanumeric characters become word separators (`_`)
- Symbols longer than 16 chars are truncated + hashed (PLASMA silent limit)

### Generated Files
`genAllFlags()` produces two files on every pack:

**`gen_flags.plh`** — included by every script module:
```plasma
const GF_KILL   = 95
const GF_ATTACK = 45
// ... one const per flag ...
const MAX_GAME_FLAG = 188
```

**`gen_flags.pla`** — loadable module with runtime name↔number lookup (used by godmode debug overlay):
```plasma
def _flags_nameForNumber(num)#1  // num → "KILL"
def _flags_numberForName(name)#1 // "KILL" → 95
```

### Runtime Storage
32-byte bitfield `ba_gameFlags` inside `TGlobal` (defined in `playtype.plh`):
```plasma
struc TGlobal
  ...
  byte[32] ba_gameFlags   // 256 bits; supports flag numbers 0–255
  ...
end
```

`setGameFlag(flagNum, val)` and `getGameFlag(flagNum)` in `gameloop.pla`:
```plasma
byteNum = flagNum >> 3        // which byte
mask    = 1 << (flagNum & 7)  // which bit
```

`getGameFlag` returns 0 or non-zero (not strict boolean) — PLASMA treats any non-zero as true.

The `TGlobal` heap struct is saved/loaded with game state via `rwGame()`.

---

## Script Compilation

### Structure
Every Mythos script is a `procedures_defreturn` block. The packer emits one PLASMA `def` per script:
```plasma
def scMyScript(v_arg1, v_arg2)
  word v_localVar     // auto-declared for every variable referenced in the body
  v_localVar = 0
  // ... compiled body ...
  return 0
end
```

Variables (`variables_get`/`variables_set` blocks) are **strictly local** to their enclosing `def`. There are no global user variables. The packer discovers variable names dynamically by scanning blocks — there is no separate variable declaration step. The `<variables>` XML element (if present in saved Blockly XML) is completely ignored.

### Block Dispatch
The packer dispatches entirely on `block.@type`. The `id`, `x`, `y`, and `inline` attributes on blocks are ignored. What matters:
- `block.@type` — drives all dispatch
- `block > field[@name]` — named scalar values
- `block > value[@name] > block` — named expression subtrees
- `block > statement[@name] > block` — named code body sequences
- `block > mutation` — `arg` children for function params; `elseif`/`else` counts for `controls_if`
- `block > next > block` — linear continuation (next statement)

### All Custom Block Types

**Flags:**
| Block type | Role | PLASMA emitted |
|---|---|---|
| `interaction_get_flag` | expression | `getGameFlag(GF_X)` |
| `interaction_set_flag` | statement | `setGameFlag(GF_X, 1)` |
| `interaction_clr_flag` | statement | `setGameFlag(GF_X, 0)` |

**Items / Players:**
| Block type | PLASMA emitted |
|---|---|
| `interaction_give_item` | `giveItemToParty(...)` |
| `interaction_take_item` | `takeItemFromParty(...)` |
| `interaction_has_item` | `partyHasItem(...)` (expr) |
| `interaction_add_player` | `addPlayerToParty(...)` |
| `interaction_remove_player` | `removePlayerFromParty(...)` |
| `interaction_bench_player` | `benchPlayer(...)` |
| `interaction_unbench_player` | `unbenchPlayer(...)` |
| `interaction_has_player` | `partyHasPlayer(...)` (expr) |

**Stats:**
| Block type | PLASMA emitted |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [badvision/lawless-legends](https://github.com/badvision/lawless-legends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
