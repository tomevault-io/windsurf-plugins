---
trigger: always_on
description: You are assisting with writing CLEO scripts for GTA San Andreas using CLEO5 and Sanny Builder 4 syntax.
---

# CLEO5 Script Development Workspace

You are assisting with writing CLEO scripts for GTA San Andreas using CLEO5 and Sanny Builder 4 syntax.

## Critical Rules

1. **NEVER guess opcodes.** Before using ANY opcode, verify it exists by searching `reference/opcode-index.md`. If you cannot find it, tell the user rather than making one up.
2. **ALWAYS check parameter signatures.** After finding an opcode in the index, read the detailed entry in the appropriate file (class file under `reference/opcodes-by-class/` for default opcodes, or `reference/ext-{name}.md` for extension opcodes) to get exact parameter names, types, and order.
3. **Prefer named parameters for clarity.** Sanny Builder 4 supports `{paramName} value` format: `wait {time} 1000`. Positional syntax (`wait 1000`) is also valid and commonly used for simple opcodes.
4. **Every loop MUST contain `wait {time} 0`** (or higher). Loops without wait will freeze the game.
5. **Always clean up models.** After `request_model` + `load_all_models_now`, always call `mark_model_as_no_longer_needed` when done.
6. **Use enums, not magic numbers.** Check `reference/enums.md` for the correct enum values (e.g., `Fade.Out` not `0`, `Font.Menu` not `1`).
   - In raw comparisons and assignments, prefer the verified numeric value if the enum form triggers a parser error (for example `pedTypeId == 6 // PedType.Cop`).
7. **Prefer parser-safe syntax over compact syntax.** If an expression can be written either as one dense line or as 2-4 simple lines with temporaries, prefer the simpler form.
8. **Keep function state explicit.** When helper functions need to update coordinates, counters, or handles, prefer parameters and return values over assigning to script-level variables from inside the function body.
9. **Keep local arrays small.** Large arrays can exceed CLEO's local-variable budget for a scope. If you need more storage, use a smaller fixed buffer or memory-backed storage instead of a large local array.
10. **Never use commands marked `unsupported` or `nop`.** Existence in the index is not enough; inspect the detailed entry's flags.
11. **Stay inside the default target profile.** The default is GTA SA PC 1.0 + CLEO 5.4 and its bundled extensions. CLEO+, NewOpcodes, SAMPFUNCS, Sphere, clipboard, and ImGui are opt-in and require an explicit `{$USE ...}` directive and a stated runtime dependency.
12. **Validate every completed script.** Run `python3 tools/validate_workspace.py path/to/script.txt`. If Sanny Builder is available, compile it and repair all errors before calling it complete.

## How to Look Things Up

### Finding an opcode
1. Search `reference/opcode-index.md` for the opcode name or ID. For structured output, use `python3 tools/opcode_lookup.py search "query"` and `python3 tools/opcode_lookup.py show OPCODE_NAME` against the same canonical source.
2. The index tells you which **class** and **extension** it belongs to
3. Read the detailed file:
   - If the Extension column is `default` and Class is filled: read `reference/opcodes-by-class/{ClassName}.md`
   - If the Extension column is `default` and Class is blank: read `reference/opcodes-by-class/_General.md`
   - If the Extension column is anything else (CLEO, audio, file, input, etc.): read `reference/ext-{ExtensionName}.md` (note: CLEO+ maps to `ext-CLEOPlus.md`)

### Finding opcodes by category
- Vehicle operations: `reference/opcodes-by-class/Car.md`
- Character/ped operations: `reference/opcodes-by-class/Char.md`
- Player-specific: `reference/opcodes-by-class/Player.md`
- Camera control: `reference/opcodes-by-class/Camera.md`
- World/environment: `reference/opcodes-by-class/World.md`
- Math functions: `reference/opcodes-by-class/Math.md`
- HUD/display: `reference/opcodes-by-class/Hud.md` and `reference/opcodes-by-class/Text.md`
- Audio: `reference/opcodes-by-class/Audio.md` and `reference/ext-audio.md`
- Objects: `reference/opcodes-by-class/Object.md`
- Tasks/AI: `reference/opcodes-by-class/Task.md`
- Game state: `reference/opcodes-by-class/Game.md`
- Flow control / variables / comparisons: `reference/opcodes-by-class/_General.md`

### CLEO-specific opcodes (0A8C+)
- Core CLEO: `reference/ext-CLEO.md` (memory ops, script loading, dynamic libraries)
- CLEO+: `reference/ext-CLEOPlus.md` (extended opcodes)
- File I/O: `reference/ext-file.md`
- Input: `reference/ext-input.md`
- Audio: `reference/ext-audio.md`
- Text/drawing: `reference/ext-text.md`
- Memory: `reference/ext-memory.md`
- Math: `reference/ext-math.md`
- Debug: `reference/ext-debug.md`
- INI files: `reference/ext-ini.md`
- ImGui: `reference/ext-imgui.md`

### Enum values
- All enums: `reference/enums.md`
- Common ones: `WeaponType`, `PedType`, `KeyCode`, `Font`, `Fade`, `BlipColor`, `CameraMode`, `SwitchType`, `AudioStreamAction`, `BodyPart`, `PickupType`

### Script syntax
- Full syntax reference: `reference/syntax-guide.md`

### SDK/Plugin development
- C++ SDK API: `reference/sdk-api.md`

## Workspace Structure

```
cleo-workspace/
├── AGENTS.md                    # This file (read first)
├── reference/
│   ├── opcode-index.md          # Searchable index of ALL 3,739 opcodes
│   ├── syntax-guide.md          # CLEO script syntax reference

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dryxio/cleo-ai](https://github.com/Dryxio/cleo-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
