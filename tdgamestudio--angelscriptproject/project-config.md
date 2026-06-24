---
trigger: always_on
description: - This file is guidance for AI agents working in `AngelscriptProject`.
---

# AGENTS.md

## Project Overview

- This file is guidance for AI agents working in `AngelscriptProject`.
- The primary goal is not to extend a regular game project, but to organize, verify, and solidify `Plugins/Angelscript` as a standalone, reusable Angelscript plugin for Unreal Engine. This repository serves as the host project for plugin development and validation; the real deliverable is the `Angelscript` plugin itself.
- The plugin is **no longer in prototype or foundation-building phase**. It has entered a maturity stage where the core runtime, editor integration, and test infrastructure are established, but external delivery entry points and several key capability closures still need attention.
- Current baseline: `AngelscriptRuntime` / `AngelscriptEditor` / `AngelscriptTest` three-UE-module structure is stable, with `121` `Bind_*.cpp` files, `27+` CSV state export tables, `1518+` automation test definitions across `430` test `.cpp` files, `DebugServer V2` protocol, `CodeCoverage`, `StaticJIT`, and `BlueprintImpact Commandlet` all landed. GameplayTags support now lives in the optional `AngelscriptGameplayTags` plugin, while `AngelscriptGAS` depends on it for GAS-facing integration. Only `2` tests remain Disabled (both `#ue57-headless` known limitations).
- AngelScript base version is `2.33 + selective 2.38 compatibility`; the fork has diverged too far for a wholesale upgrade — the strategy is to selectively absorb improvements from higher versions. See `Documents/Guides/AngelscriptForkStrategy.md`.
- `Plugins/Angelscript/` is the core workspace. The vast majority of implementation, fixes, cleanup, and tests should land here first. `Source/AngelscriptProject/` retains only the minimal host project content — do not push plugin logic back into the project module unless the task explicitly requires it.

## Project Directory Structure

```
AngelscriptProject/
├── AGENTS.md                                # AI guidance (EN) — this file
├── AGENTS_ZH.md                             # AI guidance (ZH)
├── CLAUDE.md                                # Redirect → AGENTS.md
│
├── Plugins/Angelscript/                     # ★ Core deliverable (1619 files)
│   ├── README.md                            # Plugin README for consumers
│   ├── Angelscript.uplugin
│   └── Source/
│       ├── AngelscriptRuntime/              # Runtime module (209 .cpp)
│       │   ├── Core/                        # Engine core, type system, compilation
│       │   ├── Binds/                       # 121 Bind_*.cpp (engine API bindings)
│       │   ├── ClassGenerator/              # Dynamic class gen, hot reload, versioning
│       │   ├── Debugging/                   # DebugServer V2 (DAP protocol)
│       │   ├── StaticJIT/                   # Static JIT compilation
│       │   ├── Preprocessor/                # Script preprocessor (#include, #if)
│       │   ├── FunctionLibraries/           # 21 mixin helper libraries
│       │   ├── Subsystem/                   # Script subsystem base classes
│       │   ├── Dump/                        # 27+ CSV state export tables
│       │   ├── CodeCoverage/                # Per-line coverage tracking
│       │   ├── Testing/                     # Runtime test support
│       │   └── ThirdParty/                  # AngelScript 2.33 vendored source
│       ├── AngelscriptEditor/               # Editor module (49 .cpp)
│       │   ├── HotReload/                   # File watcher & class reinstancing
│       │   ├── CodeGen/                     # Editor-time code gen for IDE
│       │   ├── BlueprintImpact/             # BP change scanner & commandlet
│       │   ├── SourceNavigation/            # Jump-to-source support
│       │   └── ContentBrowser/              # .as files in Content Browser
│       ├── AngelscriptTest/                 # Test module (430 .cpp, 28+ themes)
│       └── AngelscriptUHTTool/              # UHT C# code gen toolchain
│
├── Plugins/AngelscriptGameplayTags/         # Optional GameplayTags extension plugin
│   ├── Source/
│   │   ├── AngelscriptGameplayTags/         # Runtime GameplayTag bindings and replay
│   │   ├── AngelscriptGameplayTagsEditor/   # GameplayTag change listener and reload bridge
│   │   └── AngelscriptGameplayTagsTest/     # GameplayTags-specific automation tests
│
├── Source/                                  # Host project (minimal, 8 files)
├── Script/                                  # AngelScript examples (37 .as)
│   ├── Examples/                            # Core / EnhancedInput / Extended
│   ├── Automation/                          # Script automation entry
│   └── Tests/                               # Script-level tests
│
├── Reference/
│   └── README.md                            # Repo index, pull cmds, priorities
│
├── .agents/skills/
│   └── README.md                            # OpenSpec workflow & skill guide
│
├── openspec/                                # ★ Active change lifecycle (48 files)
│   ├── changes/                             # In-progress & archived changes
│   └── specs/                               # Shared specifications
│
├── Documents/
│   ├── Guides/
│   │   ├── Build.md                         # Build commands & execution
│   │   ├── Test.md                          # Test runner & suite usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TDGameStudio/AngelscriptProject](https://github.com/TDGameStudio/AngelscriptProject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
