---
trigger: always_on
description: Provides compatibility properties (`_currentSectionIndex`, `_currentItemIndex`, `_currentSubItemIndex`) that map to MenuBase's `_indices` array. All building navigators in `Navigators/` extend this class.
---

# CLAUDE.md

BepInEx 5 accessibility mod for "Against the Storm" (roguelite city-builder by Eremite Games) — screen reader support via Prism (`prism.dll`, vendored under `prism/native/win-x64/`). Uses HarmonyX patching and reflection against `Assembly-CSharp.dll` (namespace `Eremite`).

## Game Overview

Against the Storm is a roguelite city-builder. Players act as a Viceroy building settlements on a rectangular tile grid, managing resources, buildings, workers (7 species), and recipes. The win condition is filling the Reputation bar before Impatience fills. Each year has 3 seasons (Drizzle, Clearance, Storm). The overworld is a hex-grid World Map centered on the Smoldering City, with meta-progression via Seals, Deeds, and Capital upgrades. The game is fundamentally mouse-driven with no native screen reader or keyboard navigation support — this mod provides both. See `llm-docs/` for detailed game mechanics and API reference.

## Build & Deploy

```powershell
powershell -ExecutionPolicy Bypass -File build.ps1                          # Release build + deploy to game folder
powershell -ExecutionPolicy Bypass -File build.ps1 -Configuration Debug     # Debug build + deploy
```

**Note**: Run from the repo root. `-ExecutionPolicy Bypass` is required when running from bash.

For release packaging, see `RELEASE-INSTRUCTIONS.md`.

## Changelog

`changes.md` is the full project changelog, ordered latest version first. The top section (`## Changes since vX.Y.Z`) tracks unreleased changes. After each commit, append a one-line summary to the appropriate subsection there (New features / Bug fixes / Internal). Keep entries concise and user-facing where possible. On release, rename that section to the new version number and add a fresh `## Changes since vX.Y.Z` section above it. **Always include `changes.md` in the commit itself** — do not leave it as a separate follow-up.

## Key Locations

- **Source**: `ATSAccessibility/`
- **Game reference**: `game-source/` (read-only decompiled)
- **Debug log**: `%USERPROFILE%\AppData\LocalLow\Eremite Games\Against the Storm\Player.log` - check first for `[ATSAccessibility]` output
- **Game API reference**: `llm-docs/game-api-reference.md` — reflected game types, services, and members

## Code Organization

The codebase is organized into subdirectories by responsibility:

```
ATSAccessibility/
├── Core/        - Entry point, managers, base classes, interfaces
├── Overlays/    - Game popup navigation (*Overlay.cs, UINavigator, EncyclopediaNavigator)
├── Reflection/  - Game API access via reflection (*Reflection.cs, ReflectionHelper)
├── Handlers/    - Key handlers, mode controllers (*Handler.cs, *Controller.cs, MapNavigator)
├── Navigators/  - Building navigators + BuildingSectionNavigator base class
├── Utils/       - Utilities, formatters, readers, scanners, helpers
├── Panels/      - Information panels and menu hubs (*Panel.cs, MenuHub)
```

**Core/** (`ATSAccessibility.Core`): Entry point (`Plugin.cs`, `AccessibilityCore.cs`), managers (`KeyboardManager.cs`, `PopupRouter.cs`), base classes (`MenuBase.cs`), interfaces (`IKeyHandler.cs`, `IBuildingNavigator.cs`, `IHelpProvider.cs`), input handling (`InputPatches.cs`, `InputBlocker.cs`), help system (`HelpCollector.cs`), scene constants (`SceneConstants.cs`).

**Reflection/** (`ATSAccessibility.Reflection`): Game API access — one `*Reflection.cs` per game system. Core access via `GameReflection.cs`. All use `ReflectionHelper` for null-safe field/property/method access.

**Overlays/** (`ATSAccessibility.Overlays`): Popup navigation — one `*Overlay.cs` per game popup. Most extend `MenuBase`; `UINavigator`, `EncyclopediaNavigator`, and `MetaRewardsOverlay` implement `IKeyHandler` directly.

**Handlers/** (`ATSAccessibility.Handlers`): Key handlers and mode controllers. `MapNavigator.cs` and `WorldMapNavigator.cs` handle map-level navigation. `TutorialTooltipHandler.cs` auto-announces tutorial tooltips.

**Navigators/** (`ATSAccessibility.Navigators`): Building navigators — one per building type. All extend `BuildingSectionNavigator` (also in this directory). Includes shared helpers `BuildingWorkerSection.cs` and `BuildingUpgradesSection.cs`.

**Utils/** (`ATSAccessibility.Utils`): `Speech.cs`, `SoundManager.cs`, `EventAnnouncer.cs`, `TypeAheadSearch.cs`, `*Helper.cs`, `*Reader.cs`, `*Scanner.cs`, `FormattingUtils.cs`, `NavigationUtils.cs`.

**Panels/** (`ATSAccessibility.Panels`): Information panels (`*Panel.cs`), `MenuHub.cs`, `ConfirmationDialog.cs`, `HelpOverlay.cs` (F12 context-sensitive help).

---

## Design Patterns

### 1. Key Handler Pattern (IKeyHandler)

Priority chain where first active handler consumes the key.

```csharp
public class MyHandler: IKeyHandler {
	public bool IsActive => /* side-effect free check */;

	public bool ProcessKey(KeyCode keyCode, KeyboardManager.KeyModifiers modifiers) {
		switch (keyCode) {
			case KeyCode.UpArrow:
				DoSomething();
				return true;
			case KeyCode.Escape:
				// Pass to game to close popup
				return false;
			default:
				// Consume all other keys while active
				return true;
		}
	}
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rashadnaqeeb/ats-accessibility-mod](https://github.com/rashadnaqeeb/ats-accessibility-mod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
