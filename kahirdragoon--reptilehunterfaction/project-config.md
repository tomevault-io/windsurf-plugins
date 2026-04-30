---
trigger: always_on
description: A RimWorld 1.6 mod adding the "Reptile Hunters" faction—spacer-tech alien raiders who kidnap pawns through specialized extraction buildings and drug trafficking. Combines Harmony patching, procedural settlement generation, and custom building mechanics.
---

# Reptile Hunter Faction Mod - AI Coding Guide

## Project Overview
A RimWorld 1.6 mod adding the "Reptile Hunters" faction—spacer-tech alien raiders who kidnap pawns through specialized extraction buildings and drug trafficking. Combines Harmony patching, procedural settlement generation, and custom building mechanics.

**Key Tech**: C# (.NET 4.8.1), Harmony 2.4.2, RimWorld 1.6 API, XML definitions
Use the latest language version features

### Workspace Structure
This workspace contains both the mod source and the RimWorld engine reference:
- **Hunter Faction** (this folder): The mod project—all modifications go here
- **RimWorld Source** (sibling workpad): Decompiled RimWorld 1.6 source for API reference **only**. **Never modify this folder**—it exists purely for understanding RimWorld internals when implementing patches and custom logic

## Architecture Essentials

### Mod Entry & Initialization
- **[ReptileHunterFactionMod.cs](Source/ReptileHunterFaction/ReptileHunterFactionMod.cs)**: Single Harmony patcher initialization with namespace `ReptileHunterFaction`
- **[ReptileHunterFactionDefOf.cs](Source/ReptileHunterFaction/ReptileHunterFactionDefOf.cs)**: DefOf static class—register all XML defs here for compile-time safety (e.g., `RHF_ReptileHunters`, `RHF_SBD_Extractor`)
- All game hooks via **Harmony/** patches, never direct mod hooks

### Procedural Base Generation (BaseGen Framework)
[GenStep_RHFSettlement.cs](Source/ReptileHunterFaction/GenStep_RHFSettlement.cs) orchestrates settlement layout:
- Uses RimWorld's BaseGen symbol stack (push `"settlement"`, `"rhf_mineDefense"`, `"rhf_autocannonDefense"`)
- Creates random 40-60 cell rectangular areas, configures global mine/prison/druglab limits
- References SymbolResolvers to inject custom rooms (see pattern in `SymbolResolver_Interior_*.cs`)
- **Key pattern**: `BaseGen.globalSettings`, `BaseGen.symbolStack.Push()`, `BaseGen.Generate()`, post-process with `MapGenUtility.PostProcessSettlement()`

### Custom Building: Extractor
[Building_Extractor.cs](Source/ReptileHunterFaction/Building_Extractor.cs) (408 lines):
- Converts captured pawns into "insect blood" via genes—core mod mechanic
- Uses `Building_Enterable` (compatible with shuttle bases in Odyssey)
- Manages internal state via [ExtractorState.cs](Source/ReptileHunterFaction/ExtractorState.cs)
- Broadcasts effects/sounds; integrates with job system for gameplay feedback

### Harmony Patches (In [Harmony/](Source/ReptileHunterFaction/Harmony/) folder)
- **Patch_PawnGenerator_GeneratePawn_Genes.cs**: Post-patch pawn spawn to inject faction/pawnkind genes via `SpawnGenesExtension` ModExtension
- **Patch_Settlement_MapGeneratorDef.cs**: Override settlement generator for this faction
- **Patch_GlobalSetting_Clear.cs**: Ensure BaseGen global state reset between settlements
- **Patch_PawnGenerator_RemoveSBDFromHuners.cs**: Exclude Super Bug Drug from faction pawns (SBD = faction's own drug)

### XML Definition Structure ([Defs/](Defs/) folder)
- **Faction.xml**: FactionDef `RHF_ReptileHunters` with raid curves, damage effects, cannibal/sadist meme requirements
- **Pawnkinds.xml**: Race-specific raider pawnkinds (linked via `SpawnGenesExtension`)
- **DrugMedicine.xml, SuperBugDrug.xml**: Economic mechanics
- **SettlementMapGenerator.xml**: Registry for custom map generation
- **Patches/**: XML patches for mod compatibility (Odyssey, armor implants, etc.)

## Critical Developer Workflows

### Building & Testing
```bash
# Build command (from Source/ folder)
dotnet build ReptileHunterFaction.csproj -c 1.6
```
- Output: `1.6/Assemblies/ReptileHunterFaction.dll`
- Configuration is hardcoded to "1.6" in `.csproj`; no Debug/Release variants for game mods

### Adding New DefOfs
1. Create/modify XML in [Defs/](Defs/) (e.g., `<ThingDef defName="RHF_MyThing">`)
2. Register static field in [ReptileHunterFactionDefOf.cs](Source/ReptileHunterFaction/ReptileHunterFactionDefOf.cs)
3. Rebuild DLL—RimWorld's DefOfHelper auto-populates at game startup

### SymbolResolver Pattern (Procedural Rooms)
Reference [SymbolResolver_BasePart_Indoors_Leaf_ExtractionRoom.cs](Source/ReptileHunterFaction/SymbolResolver_BasePart_Indoors_Leaf_ExtractionRoom.cs):
- Inherit `SymbolResolver_BasePart_Indoors_Leaf`
- Override `Resolve()` to place buildings/furniture
- Register symbol in XML: `<SymbolResolverDef><symbol>rhf_extractionRoom</symbol></SymbolResolverDef>`
- Push to stack in GenStep: `BaseGen.symbolStack.Push("rhf_extractionRoom", resolveParams)`

## Common Patterns & Conventions

### ModExtensions for Pawn Gene Injection
[SpawnGenesExtension.cs](Source/ReptileHunterFaction/SpawnGenesExtension.cs):
```xml
<modExtensions>
  <li Class="ReptileHunterFaction.SpawnGenesExtension">
    <genes>
      <li>AG_InsectFlesh</li>
    </genes>
  </li>
</modExtensions>
```
Applied to `FactionDef` and `PawnKindDef` to auto-inject genes during pawn spawn.

### Hardcoded Gene Fallbacks
[Building_Extractor.cs](Source/ReptileHunterFaction/Building_Extractor.cs) line ~19:
```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kahirdragoon/ReptileHunterFaction](https://github.com/kahirdragoon/ReptileHunterFaction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
