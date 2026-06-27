---
trigger: always_on
description: Dalamud FFXIV plugin. Replaces the game's default floating combat text with WoW-style flashy numbers.
---

# FloatyText — Developer Notes

Dalamud FFXIV plugin. Replaces the game's default floating combat text with WoW-style flashy numbers.

## Build

```powershell
cd D:\Projects\FloatyText
dotnet build
# Output: bin\Debug\net10.0-windows\FloatyText.dll
# Load in-game: Dalamud Settings → Experimental → Dev Plugin Locations → add the DLL path
# Reload after rebuild: /xlplugins → disable/re-enable, or /xldev → reload
```

## Project structure

| File | Role |
|---|---|
| `Plugin.cs` | Entry point. Service injection, WindowSystem, Framework.Update hook. Creates FontManager, FloatyTextRenderer, CombatEventHandler. |
| `CombatEventHandler.cs` | Two spawn paths. **Instant mode** (default): spawns at packet time from `ActionEffectHook.EffectReceived` — classifies from packet flags, looks up action name/icon via Lumina `Action` sheet, filters to own/pet actions + things targeting the player, remembers spawned values; `FlyTextCreated` then suppresses the game's delayed duplicate by value-match (3s window). Unmatched fly text (DoT/HoT ticks, misses — ActorControl, not ActionEffect) spawns via the classic path. **Classic mode**: packet effects queue locally; `FlyTextCreated` dequeues for position/filter and spawns (synced to hit animations). |
| `ActionEffectHook.cs` | Unsafe hook via `FFXIVClientStructs.ActionEffectHandler.Receive`. Fires `EffectReceived(PendingEffect)` at network-packet time, once per damage/heal entry per target (iterates AoE targets via the effectTrail ulong array; 8 effect entries per target). EffectTypes 3/5/6 = damage (5 blocked, 6 parried), 4 = heal. Decodes >65535 values: `Value + (Param2 << 16)`. All pointers null-checked before deref (AVs in unsafe code are NOT catchable). |
| `ScreenPositionResolver.cs` | Projects world → screen. Uses **local player's Y** (not the enemy's) so text stays at eye-level even for raid bosses that fill the screen. Uses `IObjectTable.SearchByEntityId(uint)` (NOT `SearchById(ulong)`) for packet-level 32-bit entity IDs. |
| `FloatyTextRenderer.cs` | Manages active entries, draws via `ImGui.GetBackgroundDrawList()`. One font handle per family (large pre-baked size); renders user-specified size as a downscale → crisp. Draw order: shadow → crit glow → outline → main text → ability icon → ability name. Entry list guarded by `_entriesLock` — Spawn/Tick run on the framework thread, Draw on the render thread. |
| `FloatingTextEntry.cs` | Per-number animation state. Ease-out quadratic float, overshoot-bounce crit pop (2.0× → 1.12× → 1.0×), decaying sinusoidal shake for crits. Exposes `DisplayPosition = Position + ShakeOffset`. |
| `FontManager.cs` | Holds one `IFontHandle` per game-font family, loaded via `FontAtlas.NewGameFontHandle`. Large source size → user sizes are downscales → no bitmap blur. |
| `Configuration.cs` | `IPluginConfiguration`. Saved by Dalamud automatically. |
| `ConfigWindow.cs` | Extends `Dalamud.Interface.Windowing.Window`. Registered with `WindowSystem`. Tabbed layout (`ImGui.BeginTabBar`): Visibility / Colors / Text & Font / Animation, Save buttons below the tab bar. All ImGui property edits use local-copy pattern (can't `ref` auto-properties). |

## Dalamud v15 API notes (important)

| Topic | Detail |
|---|---|
| Target framework | `net10.0-windows` — Dalamud 15 is built on .NET 10 |
| ImGui namespace | `Dalamud.Bindings.ImGui` (was `ImGuiNET`). Reference `Dalamud.Bindings.ImGui.dll`, NOT `ImGui.NET.dll` |
| `Plugin.Name` | Removed — plugin name comes from the manifest (`FloatyText.json`) |
| `IFlyTextGui` | Inject via `[PluginService]` — `PluginInterface.Create<>()` is gone |
| `IClientState.LocalPlayer` | **Gone**. Local player is now `IObjectTable.LocalPlayer` |
| `IObjectTable.SearchByEntityId(uint)` | Use this for 32-bit packet entity IDs. `SearchById(ulong)` is a different overload for 64-bit game object IDs. |
| `FlyTextKind` enum | **Completely renamed** in v15. Old→New: `AutoAttack`→`Damage`, `CriticalHit`→`DamageCrit`, `DirectHit`→`DamageDh`, `CriticalDirectHit`→`DamageCritDh`, `NamedAttack`→`Named`, `Healing`→`Healing` (same), `CriticalHealingBuff`→`HealingCrit`, `Miss`→`Miss` (same). Also new: `AutoAttackOrDot`, `AutoAttackOrDotCrit`, `AutoAttackOrDotDh`, `AutoAttackOrDotCritDh`, `Dodge`. |
| `BattleNpcSubKind` enum | Renamed in v15: `Enemy`→`Combatant`, `Chocobo`→`RaceChocobo`. Full set: `None, BNpcPart, Pet, Buddy, Player, Combatant, RaceChocobo, LovmMinion, NpcPartyMember`. `StatusFlags` (`PartyMember`, `AllianceMember`, `Hostile`, …) unchanged. |
| `ITextureProvider` icons | `GetFromGameIcon(new GameIconLookup(iconId))` → `ISharedImmediateTexture` → `.GetWrapOrDefault()` (null until loaded — skip that frame). Cached by Dalamud, safe to call per-frame at draw time. Draw via `drawList.AddImage(wrap.Handle, min, max, uv0, uv1, tint)`. `IDalamudTextureWrap` is in `Dalamud.Interface.Textures.TextureWraps`. **Gotchas:** FlyText sends `uint.MaxValue` (-1) as well as 0 for "no icon", and `GetFromGameIcon` THROWS `IconNotFoundException` for invalid IDs (it doesn't return null) — normalize the sentinel and try/catch the lookup. |
| `MathF.Clamp / MathF.Lerp` | **Removed from .NET 10**. Use `float.Clamp` / `float.Lerp` instead. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BoujeeBecky/FloatyText](https://github.com/BoujeeBecky/FloatyText) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
