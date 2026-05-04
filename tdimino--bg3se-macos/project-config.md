---
trigger: always_on
description: macOS port of Norbyte's Script Extender for Baldur's Gate 3. Goal: feature parity with Windows BG3SE.
---

# BG3SE-macOS

macOS port of Norbyte's Script Extender for Baldur's Gate 3. Goal: feature parity with Windows BG3SE.

**Version:** v0.36.50 | **Parity:** ~94% | **Target:** Full Windows BG3SE mod compatibility

## Stack

- C17/C++20, Universal binary (arm64 + x86_64)
- Dobby (inline hooking), Lua 5.4, lz4, zlib
- Injection: `insert_dylib` static Mach-O patching (LC_LOAD_WEAK_DYLIB) — DYLD_INSERT_LIBRARIES is dead (crashes through Steam)
- Launcher bypass: `defaults write com.larian.bg3 NoLauncher 1` (set automatically by harness)

## Structure

- `src/injector/main.c` - Core injection, hooks, Lua state, Osi dispatch
- `src/core/crashlog.c` - Crash-resilient logging (mmap ring buffer, signal handler, breadcrumbs)
- `src/lua/lua_*.c` - Ext.* API implementations
- `src/osiris/` - Osiris types, function cache, handle encoding, pattern scanning
- `src/stats/` - RPGStats system + prototype managers
- `src/entity/` - Entity Component System (GUID lookup, components)
- `ghidra/offsets/` - Reverse-engineered offsets documentation

## Modding Toolkit (36 Commands)

```bash
# Core pipeline
PYTHONPATH=tools python3 -m bg3se_harness status            # Game/socket/patch state
PYTHONPATH=tools python3 -m bg3se_harness launch --continue # Autonomous: auto-loads most recent save
PYTHONPATH=tools python3 -m bg3se_harness test [filter]     # Full pipeline: build+patch+launch+continue+test → JSON

# Game inspection
PYTHONPATH=tools python3 -m bg3se_harness entity <GUID> [--component X]  # Inspect entity
PYTHONPATH=tools python3 -m bg3se_harness stats <name> [--diff OTHER]    # RPG stats + comparison
PYTHONPATH=tools python3 -m bg3se_harness components [--namespace eoc]   # List 1,999+ component types
PYTHONPATH=tools python3 -m bg3se_harness probe <0xADDR> [--classify]    # Memory inspection

# Development
PYTHONPATH=tools python3 -m bg3se_harness run "<lua>"       # Inline Lua
PYTHONPATH=tools python3 -m bg3se_harness eval script.lua   # File/stdin Lua (piping)
PYTHONPATH=tools python3 -m bg3se_harness watch script.lua  # Hot-reload on save
PYTHONPATH=tools python3 -m bg3se_harness screenshot        # Game window (1568px, JPEG, Claude Code safe)
PYTHONPATH=tools python3 -m bg3se_harness dump spells       # Bulk extract game data
PYTHONPATH=tools python3 -m bg3se_harness events --subscribe SessionLoaded  # Stream events (JSONL)

# Diagnostics
PYTHONPATH=tools python3 -m bg3se_harness crashlog          # Parse crash ring buffer (no socket)
PYTHONPATH=tools python3 -m bg3se_harness benchmark "Ext.Stats.Get('WPN_Longsword')"  # Perf measurement
PYTHONPATH=tools python3 -m bg3se_harness diff-test base.json curr.json   # Test regression comparison

# Mod management (delegates to BG3MacModManager or pure Python fallback)
PYTHONPATH=tools python3 -m bg3se_harness mod list          # Installed mods + enabled/SE status
PYTHONPATH=tools python3 -m bg3se_harness mod install <path.pak>  # Install local PAK
PYTHONPATH=tools python3 -m bg3se_harness mod enable <name> # Enable in modsettings.lsx
PYTHONPATH=tools python3 -m bg3se_harness mod search <query>  # Search Nexus Mods API

# Web integrations (Nexus + bg3.wiki — stdlib urllib, 24h file cache)
PYTHONPATH=tools python3 -m bg3se_harness mod changelog <id>       # Nexus changelogs (HTML stripped, newest-first)
PYTHONPATH=tools python3 -m bg3se_harness mod versions <id>        # Nexus file list (file_id, version, category, size)
PYTHONPATH=tools python3 -m bg3se_harness mod updated --period 1w  # Recently-updated mods (1d/1w/1m)
PYTHONPATH=tools python3 -m bg3se_harness wiki spell "Fireball"    # Parsed {{Feature page}} template fields
PYTHONPATH=tools python3 -m bg3se_harness wiki item "Longsword +1" # Parsed {{WeaponPage}} / {{ArmourPage}} fields
PYTHONPATH=tools python3 -m bg3se_harness wiki verify "Fireball" --expect-uid Projectile_Fireball  # Offline uid cross-check
PYTHONPATH=tools python3 -m bg3se_harness wiki clear-cache         # Wipe ~/.config/bg3se-harness/wiki_cache/

# Parity + compatibility + diagnostics
PYTHONPATH=tools python3 -m bg3se_harness parity scan       # Compare Ext table vs Windows baseline
PYTHONPATH=tools python3 -m bg3se_harness parity missing    # List gaps (offline)
PYTHONPATH=tools python3 -m bg3se_harness compat list       # Available test scenarios
PYTHONPATH=tools python3 -m bg3se_harness compat run mcm    # Autonomous mod compat test
PYTHONPATH=tools python3 -m bg3se_harness doctor            # Verify all prerequisites
PYTHONPATH=tools python3 -m bg3se_harness save list         # Available saves with metadata
PYTHONPATH=tools python3 -m bg3se_harness author new MyMod  # Scaffold new mod

# Menu automation (Vision OCR + CGEvent click)
PYTHONPATH=tools python3 -m bg3se_harness menu detect       # OCR main menu buttons → JSON
PYTHONPATH=tools python3 -m bg3se_harness menu click "Continue"  # Click button by name

# RE + flags
PYTHONPATH=tools python3 -m bg3se_harness flags             # 40 discovered game CLI flags
PYTHONPATH=tools python3 -m bg3se_harness ghidra decompile <name|0xADDR>  # Ghidra RE bridge
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tdimino/bg3se-macos](https://github.com/tdimino/bg3se-macos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
