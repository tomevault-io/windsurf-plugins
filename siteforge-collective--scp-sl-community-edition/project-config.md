---
trigger: always_on
description: <!-- Companion docs — read these when working on the relevant system: -->
---

<!-- Companion docs — read these when working on the relevant system: -->
<!-- CLAUDE-systems.md       — detailed subsystem descriptions, role tables, ScriptableObjects -->
<!-- CLAUDE-firearm.md       — full weapon system: architecture, all bug fixes (sessions 2-6), animation, invariants -->
<!-- CLAUDE-scenes.md        — all 6 scenes: root hierarchies, Facility GameManager/camera stack (ViewmodelCamera!), HUD canvases -->
<!-- CLAUDE-prefabs-items.md — item/weapon prefabs: Defined Items (Resources), viewmodel/pickup/thirdperson triplets -->
<!-- CLAUDE-prefabs-roles.md — Player.prefab (ReferenceHub + all status effects), Defined Roles, models, ragdolls, SCP HUDs -->
<!-- CLAUDE-prefabs-world.md — doors, structures, lockers, elevators, targets, UI element prefabs, network-spawnable list -->
<!-- CLAUDE-binary-recovery.md — HOW TO READ GROUND TRUTH from the original build: field offsets (*_metadata.txt), ISIL decoding, Tools/ReadGameAssembly.ps1 for binary constants. READ BEFORE restoring any decompiled method — never guess constants/logic. -->
<!-- CLAUDE-workflow.md    — MANDATORY working method: the debugging loop, catalog of 18 known rip failure patterns with symptom heuristics, editing/verification rules. READ FIRST when fixing any bug in this project. -->
# CLAUDE.md — SCP: Secret Laboratory 12.0.2 Restoration Project

## Project Overview

**Product Name:** SCPSL (SCP: Secret Laboratory)
**Company:** Northwood (original game developer)
**Project Type:** Multiplayer horror/action game — restoration/reverse-engineering of SCP:SL v12.0.2
**Unity Version:** 6000.3.10f1 (Unity 6)
**Game Version:** 12.0.2 (Development build, versioned as `12.0.2-12.0.1-rc-2298ba84`)
**Target Platform:** PC (Windows primary, Linux server support)
**Default Resolution:** 1920x1080 (Full HD)

This is an open educational project that reverse-engineers and rebuilds SCP: Secret Laboratory version 12.0.2 for study purposes. The game is a multiplayer asymmetric horror game set in an SCP containment facility, where players take on roles as SCPs, D-Class personnel, scientists, MTF, or Chaos Insurgency.

---

## Development Philosophy (IMPORTANT — READ BEFORE MAKING CHANGES)

**Exact restoration is NOT required.** The goal is a working, maintainable, and enjoyable game — not a byte-perfect replica of v12.0.2.

### Allowed and encouraged:
- **Use v13 code (`CODES/SCP13.0/`) freely** for logic, architecture, and patterns — it is the same codebase evolved. Better code from v13 is preferred over broken stubs from v12 decompilation.
- **Improve readability and maintainability** — cleaner code is a better goal than matching the decompiled v12 style.
- **Add null guards, safety checks, and convenience helpers** that were missing in v12. Stability beats purity.
- **Adopt v13 patterns** (additive Weight instead of Lerp, proper OnActivated/OnShutdown transitions, etc.) even when v12 had simpler or buggier versions.
- **Keep the old PostProcessing stack** (`com.unity.postprocessing 3.5.4`, `PostProcessVolume`, `PostProcessProfile`) — the project does NOT use URP/HDRP. v13 uses `UnityEngine.Rendering.Volume`; adapt its logic but keep the old API types.
- **Future-proof the code** for modding and extension where it costs nothing.

### NOT allowed:
- Do not add gameplay features, balance changes, or new content that did not exist in v12.
- Do not break the Mirror networking contract (SyncVars, Commands, RPCs).
- Do not introduce Unity package upgrades without explicit user approval.

---

## Repository Structure

```
StabelProjectSCPSL12/
├── Assets/
│   ├── Scripts/
│   │   ├── Assembly-CSharp/      # Main game scripts (all gameplay code)
│   │   └── Utf8Json/             # Utf8Json serialization library (source)
│   ├── Mirror/                   # Mirror networking framework (full source)
│   ├── ParrelSync/               # Multi-instance Unity editor tool
│   ├── Plugins/                  # Native DLLs and precompiled assemblies
│   ├── SC Post Effects/          # Post-processing visual effects package
│   ├── UBER/                     # UBER shader system
│   ├── Standard Assets/          # Unity standard assets
│   ├── _Scenes/                  # Game scenes (menu, facility)
│   ├── Resources/                # Runtime-loadable assets
│   └── StreamingAssets/          # Streamed data
├── CODES/
│   ├── ClientCode/               # Reference decompiled client code
│   ├── ServerCode/               # Reference decompiled server code
│   └── SCP13.0/                  # Reference code from version 13.0
├── Packages/                     # Unity Package Manager manifest
├── ProjectSettings/              # Unity project settings
├── Translations/                 # Localization files
└── README.md
```

---

## Unity Packages (manifest.json)

| Package | Version |
|---|---|
| `com.unity.ai.navigation` | 2.0.12 |
| `com.unity.nuget.newtonsoft-json` | 3.2.2 |
| `com.unity.postprocessing` | 3.5.4 |
| `com.unity.ugui` | 2.0.0 |
| `com.unity.multiplayer.center` | 1.0.1 |
| `com.unity.ide.visualstudio` | 2.0.27 |

See CLAUDE-systems.md for package descriptions.

---

## Assembly Definitions (.asmdef)

All gameplay code compiles into the default `Assembly-CSharp` assembly (no custom .asmdef for game scripts).

| Assembly | Location |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Siteforge-Collective/SCP-SL-Community-Edition](https://github.com/Siteforge-Collective/SCP-SL-Community-Edition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
