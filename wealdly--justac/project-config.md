---
trigger: always_on
description: WoW addon displaying Blizzard's Assisted Combat suggestions with keybinds. Lua + WoW API + Ace3.
---

# JustAC - AI Agent Instructions

WoW addon displaying Blizzard's Assisted Combat suggestions with keybinds. Lua + WoW API + Ace3.

## Version Detection & Compatibility

**WoW 12.0 (Midnight) compatibility layer ready** - Use version conditionals for breaking API changes:

```lua
local BlizzardAPI = LibStub("JustAC-BlizzardAPI", true)

-- Check version
if BlizzardAPI.IsMidnightOrLater() then
    -- 12.0+ code path (new/fixed API)
else
    -- Pre-12.0 code path (original API)
end
```

**When to add version conditionals:**
- 12.0 error reported → Add conditional fix
- API behavior changes between versions → Wrap in version check
- New API replaces old → Keep both paths with version guard

**See:** `Documentation/VERSION_CONDITIONALS.md` for detailed patterns and examples

## Critical Workflow

1. **NEVER guess WoW API behavior** — Verify with `/script` commands in-game or check `R:\WOW\00-SOURCE\WowUISource`
2. **Propose before implementing** — Describe changes, ask "Should I proceed?"
3. **Test with debug commands** — Use `/jac test`, `/jac modules`, `/jac formcheck` to validate changes
4. **DO NOT auto-increment versions** — Track changes in `UNRELEASED.md`, only bump version on explicit instruction
5. **DO NOT auto-build or push** — Commit changes, let user build/push manually
6. **NO AI attribution** — Never add `Co-Authored-By`, credits, acknowledgments, or any other reference to AI agents/models in commit messages, code comments, README, CHANGELOG, or any project file. All contributions are authored solely by the project owner.

## Versioning

**Semantic Versioning (MAJOR.MINOR.PATCH):**
- Current: 4.12.0
- Hotfixes: 4.5.5, 4.5.6, etc. (bug fixes only)
- Features: 4.6.0, 4.7.0, etc. (new functionality)
- Breaking: 5.0.0, 6.0.0, etc. (major rewrites)

Update in three places: `JustAC.toc`, `CHANGELOG.md`, `UNRELEASED.md`

## Architecture (Load Order Matters)

LibStub modules in `JustAC.toc` — **MUST edit in dependency order**:

```
BlizzardAPI → FormCache → MacroParser → ActionBarScanner → RedundancyFilter
                                    ↓
              SpellQueue → UI/* → DefensiveEngine → GapCloserEngine → DebugCommands → Options/SpellSearch → Options/LiveSearchPopup → Options/* → TargetFrameAnchor → KeyPressDetector → JustAC
```

| Module | Role | Key Exports | Current Version |
|--------|------|-------------|-----------------|
| `Locales/*.lua` | AceLocale-3.0 localization (9 languages) | `L` global | N/A (not LibStub) |
| `SpellDB.lua` | Static spell data (defensive, class defaults) | `GetDefaults()`, `GetSpecKey()` | v8 |
| `BlizzardAPI.lua` | Root: secret value primitives, version detection | `IsSecretValue()`, `Unsecret()`, `GetActionBarUsability()` | v33 |
| `BlizzardAPI/CooldownTracking.lua` | Local CD tracking (12.0+ secret workaround) | `IsSpellReady()`, `RegisterSpellForTracking()`, `IsSpellOnLocalCooldown()` | v6 |
| `BlizzardAPI/SecretValues.lua` | Feature availability gates, aura timing | `IsRedundancyFilterAvailable()`, `IsMidnightOrLater()` | v1 |
| `BlizzardAPI/SpellQuery.lua` | Spell info, usability, rotation API, items | `GetProfile()`, `GetSpellInfo()`, `IsSpellUsable()` | v1 |
| `BlizzardAPI/StateHelpers.lua` | Defensive/item state, health, CC immunity, target analysis | `CheckDefensiveItemState()`, `GetPlayerHealthPercent()`, `IsTargetCCImmune()` | v5 |
| `FormCache.lua` | Shapeshift form state (Druid/Rogue/etc) | `GetActiveForm()`, `GetFormIDBySpellID()` | v11 |
| `MacroParser.lua` | `[mod]`, `[form]`, `[spec]` conditional parsing | `GetMacroSpellInfo()`, quality scoring | v21 |
| `ActionBarScanner.lua` | Spell→keybind lookup, slot caching | `GetSpellHotkey()`, `GetSlotForSpell()` | v35 |
| `RedundancyFilter.lua` | Hide active buffs/forms | `IsSpellRedundant()` | v41 |
| `SpellQueue.lua` | Throttled spell queue, proc detection | `GetCurrentSpellQueue()`, blacklist | v37 |
| **UI/** | **UI rendering subsystem (5 files)** | | |
| `UI/UIHealthBar.lua` | Health bar widget | `Create()`, `Update()` | v7 |
| `UI/UIAnimations.lua` | Animation helpers (glow, flash, channel fill) | `StartAssistedGlow()`, `ShowProcGlow()`, `StartFlash()` | v11 |
| `UI/UIFrameFactory.lua` | Icon frame pool | `AcquireFrame()`, `ReleaseFrame()` | v12 |
| `UI/UIRenderer.lua` | Icon rendering + Masque integration | `RenderSpellQueue()`, frame management | v17 |
| `UI/UINameplateOverlay.lua` | Nameplate overlay rendering | `Create()`, `Destroy()`, `Update()` | v6 |
| `DefensiveEngine.lua` | Defensive spell evaluation | `EvaluateDefensives()` | v1 |
| `GapCloserEngine.lua` | Gap-closer spell suggestions (offensive queue) | `GetGapCloserSpell()`, `IsGapCloserSpell()`, `InvalidateGapCloserCache()` | v3 |
| `DebugCommands.lua` | In-game diagnostics | `/jac test`, `/jac modules` | v15 |
| **Options/** | **Modular options panel (11 files)** | | |
| `Options/SpellSearch.lua` | Shared spell search, filter state, spell list utils | `BuildSpellbookCache()`, `AddSpellToList()` | v1 |
| `Options/LiveSearchPopup.lua` | Persistent modal for spell/item selection | `Open()`, `Close()`, `IsOpen()` | v1 |
| `Options/General.lua` | General tab (display mode, layout, visibility) | `CreateTabArgs()` | v4 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wealdly/JustAC](https://github.com/wealdly/JustAC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
