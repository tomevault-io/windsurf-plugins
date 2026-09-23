---
trigger: always_on
description: This codebase is inspired by **Quake 2** (id Software). The developer is deeply familiar with Quake 2's architecture and source code. **Quake 2 is the primary reference** for all lifecycle, state communication, UI control, movement, and entity patterns. Use **Quake 3** as a secondary reference for features Q2 lacks, such as client-side UI libraries or renderer module separation.
---

# OpenWarcraft Agent Guide

## Project Context

This codebase is inspired by **Quake 2** (id Software). The developer is deeply familiar with Quake 2's architecture and source code. **Quake 2 is the primary reference** for all lifecycle, state communication, UI control, movement, and entity patterns. Use **Quake 3** as a secondary reference for features Q2 lacks, such as client-side UI libraries or renderer module separation.

## Further Reading

| Topic | File |
|-------|------|
| Architecture, engine boundaries, struct/API discipline, network contracts | [ARCHITECTURE.md](ARCHITECTURE.md) |
| Client camera samples, Euler snapshot, quat slerp | [docs/architecture/client.md](docs/architecture/client.md) |
| Server-selected presentation effects and generic effect contracts | [docs/architecture/server-selected-effects.md](docs/architecture/server-selected-effects.md) |
| Environment lighting samples, day-phase stat, per-game fill hook | [docs/architecture/environment-lighting.md](docs/architecture/environment-lighting.md) |
| Native game coordinate systems and axis migration | [AXIS.md](AXIS.md) |
| Server-authored UI payload design, limits, diagnostics, scrollbar postmortem | [docs/architecture/ui-payloads.md](docs/architecture/ui-payloads.md) |
| Client-managed gameplay windows, focus, z-order, dragging, modal input, text arenas | [docs/architecture/client-windows.md](docs/architecture/client-windows.md) |
| Test discipline, build & linking rules, MPQ fixture rules | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Agent documentation capture, placement, templates, and indexing | [docs/documentation-guide.md](docs/documentation-guide.md) |
| Diagnostic tools (mpqtool, dbctool, mdxtool, ability_audit, profiler) | [docs/diagnostic-tools.md](docs/diagnostic-tools.md) |
| Vendored Lua 5.4 and the libxml2-free XML parser (`common/tinyxml.h`) | [docs/vendored-dependencies.md](docs/vendored-dependencies.md) |
| UI screen authoring, FDF conventions, ConsoleUI, stb_fdf.h | [docs/ui-authoring.md](docs/ui-authoring.md) |
| Test-suite performance measurement and parallel execution | [docs/test-performance.md](docs/test-performance.md) |
| WoW character display, DBC/skin-section/component-texture rules | [docs/wow-character.md](docs/wow-character.md) |
| WoW grass: active paths, wind-sway math, phase decorrelation, density formula, constants | [docs/games/world-of-warcraft/GRASS_TECH.md](docs/games/world-of-warcraft/GRASS_TECH.md) |
| WoW magic schools, damage types, buffs/debuffs, CC, status effects | [docs/games/world-of-warcraft/magic-and-effects.md](docs/games/world-of-warcraft/magic-and-effects.md) |
| WoW creature types, classifications, difficulty, aggro/threat; entity architecture (think-fn dispatch, spell table, spawn budget) | [docs/games/world-of-warcraft/enemies-and-creatures.md](docs/games/world-of-warcraft/enemies-and-creatures.md) |
| WoW spawn system, WorldSafeLocs DBC, per-race selection, server→client game commands | [docs/games/world-of-warcraft/spawn-and-teleport.md](docs/games/world-of-warcraft/spawn-and-teleport.md) |
| WoW area triggers, dungeon/instance map loading, warp command, pending teleport mechanism | [docs/games/world-of-warcraft/area-triggers.md](docs/games/world-of-warcraft/area-triggers.md) |
| WoW first-login race cinematics, DBC chain, M2 camera playback lifecycle | [docs/games/world-of-warcraft/cinematics.md](docs/games/world-of-warcraft/cinematics.md) |
| WoW FrameXML loading, inheritance, anchors, natural text sizing, unresolved geometry | [docs/games/world-of-warcraft/framexml-layout.md](docs/games/world-of-warcraft/framexml-layout.md) |
| WoW quest system, server-authored dialog, AzerothCore SQL extraction, quest commands | [docs/games/world-of-warcraft/quest-ui.md](docs/games/world-of-warcraft/quest-ui.md) |
| WoW weapons, classes, combat roles, specializations | [docs/games/world-of-warcraft/weapons-and-classes.md](docs/games/world-of-warcraft/weapons-and-classes.md) |
| WoW gameplay features: implemented vs missing (WoWee gap analysis), loot system details | [docs/games/world-of-warcraft/gameplay-features.md](docs/games/world-of-warcraft/gameplay-features.md) |
| Entity sound architecture | [architecture/sound.md](architecture/sound.md) |
| WC3 background music, `Music.slk`/skin lookup, `svc_music`, optional FFmpeg streaming | [docs/games/warcraft-3/music.md](docs/games/warcraft-3/music.md) |
| WC3 data model (SLK, unit stats, combat) | [docs/wc3-data-model.md](docs/wc3-data-model.md) |
| WC3 attack damage math, runtime modifiers, armor/type multipliers, projectile impact timing | [docs/games/warcraft-3/attack-damage.md](docs/games/warcraft-3/attack-damage.md) |
| WC3 JASS native coverage, callback contracts, state ownership | [docs/games/warcraft-3/jass-native-coverage.md](docs/games/warcraft-3/jass-native-coverage.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [corepunch/open-realm](https://github.com/corepunch/open-realm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
