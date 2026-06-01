---
trigger: always_on
description: Comprehensive contributor guide for this MuOnline client project.
---

# Repository Guidelines

Comprehensive contributor guide for this MuOnline client project.

## Purpose and scope
- MuOnline client clone built on .NET 10 + MonoGame 3.8+.
- Uses Season 6 protocol; reads Season 20 (1.20.61) client data for assets.
- Targets OpenMU or any compatible S6 server; educational/research intent—do not commit proprietary data.

## CRITICAL: Automatic Pattern Enforcement

**IMPORTANT**: When working on specific task categories, you MUST automatically apply the corresponding patterns WITHOUT being asked. These are MANDATORY workflows, not optional suggestions.

### Task Category → Required Pattern Mapping

| When user asks to... | You MUST automatically... |
|---------------------|---------------------------|
| **Build/run** the project, fix build errors, setup environment | Apply muonline-build-and-run skill: Always pass `-p:MonoGameFramework=...` on Windows, verify DataPath in Constants.cs, provide exact commands, diagnose common failures (missing data, shader errors, backend init) |
| **After making code changes** (editing .cs files, adding features, fixing bugs) | Apply muonline-build-and-run skill: IMMEDIATELY verify compilation with exact build commands, test affected platform heads (DX/GL/Linux), report build status before claiming success |
| **Fix animations**, equipment rendering, helm visibility, bone attachments | Apply muonline-animation-and-equipment skill: Check HelmModelRules.cs, understand object hierarchy (WorldObject→ModelObject→WalkerObject→Player/NPC/Monster), distinguish OneShot vs Loop animations, test both DX/GL backends, check bone transforms |
| **Create UI controls**, panels, windows, HUD elements | Apply muonline-ui-design skill: Define Theme class with 5 bg depth levels + warm gold accent, implement static surface caching (RenderTarget2D), use UiDrawHelper for common patterns, ensure proper disposal, reference InventoryControl.cs as example |
| **Optimize performance**, reduce FPS drops, fix allocations, reduce draw calls | Apply muonline-rendering-performance skill: Stop per-frame allocations (no LINQ in Draw/Update), batch by texture/effect, reduce state changes, check Constants.cs quality toggles (RenderScale, UseMSAA, UseDynamicLight), measure before/after, test DX/GL parity |

### Enforcement Rules

1. **Detect task category** from user request
2. **Read the corresponding skill** from `~/.codex/skills/*/SKILL.md`
3. **Automatically apply** all patterns from the skill (don't ask permission)
4. **Follow all rules** from the skill (build commands, file structure, checklists)
5. **Announce** which skill you're applying: "Applying [skill-name] workflow for this [task type]"
6. **After ANY code changes**: ALWAYS use muonline-build-and-run skill to verify compilation before claiming task completion

**Examples:**
- User: "Build Windows version" → YOU: "Applying muonline-build-and-run workflow. Building with `-p:MonoGameFramework=MonoGame.Framework.WindowsDX`..."
- User: "Fix performance in ModelObject" → YOU: [Makes code changes] → "Applying muonline-build-and-run workflow to verify compilation. Building MuLinux target..."
- User: "Head disappears with helm" → YOU: "Applying muonline-animation-and-equipment workflow. Checking HelmModelRules.cs for helm type/class visibility rules..."
- User: "Create shop UI" → YOU: "Applying muonline-ui-design workflow. Creating control with Theme class (5 bg levels, gold accent #D4AF55) and static surface caching..."
- User: "Game runs slow" → YOU: "Applying muonline-rendering-performance workflow. Profiling for per-frame allocations in Draw/Update paths..."

**This is NOT negotiable.** If task matches a category, read and apply the corresponding skill automatically.

## High-level architecture
- Platform heads (MuWinDX, MuWinGL, MuLinux, MuMac, MuAndroid, MuIos) are thin launchers pointing to shared core.
- `Client.Main` hosts rendering, scenes, UI, networking, scheduling, and game objects.
- `Client.Data` parses MU asset formats and feeds the core.
- Windows heads must pass `MonoGameFramework` to select the correct MonoGame package and shader profile.

## Repository layout
- `Client.Main/`: Core engine and content.
- Scenes: `Client.Main/Scenes/` (`BaseScene`, `LoginScene`, `LoadScene`, `SelectCharacterScene`, `ServerConfigScene`, `GameScene`, tests).
- UI/world controls: `Client.Main/Controls/` (terrain, UI base `GameControl`, world controls).
- Objects: `Client.Main/Objects/` (player, monsters, NPCs, items, effects, particles, worlds).
- Worlds: `Client.Main/Objects/Worlds/<WorldName>` (Lorencia, Noria, Devias, Arena, Icarus, Atlans, Login, SelectWorld).
- Networking: `Client.Main/Networking/` (`PacketRouter`, `PacketBuilder`, handlers, services).
- State/logic: `Client.Main/Core/Client/` (`CharacterState`, `ScopeManager`, `PartyManager`, enums).
- Databases/utilities: `Client.Main/Core/Utilities/` (`ItemDatabase`, `SkillDatabase`, `MapDatabase`, `NpcDatabase`, `CharacterClassDatabase`, `AppearanceConfig`, `PlayerActionMapper`, `WorldInfoAttribute`, `NpcInfoAttribute`, `PacketHandlerAttribute`).
- Models: `Client.Main/Core/Models/` (`ScopeObject`, `ServerInfo`).
- Graphics/effects: `Client.Main/Graphics/`, `Client.Main/Effects/`.
- Content/shaders: `Client.Main/Content/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bernatvadell/muonline](https://github.com/bernatvadell/muonline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
