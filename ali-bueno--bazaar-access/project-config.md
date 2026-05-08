---
trigger: always_on
description: A BepInEx mod making "The Bazaar" accessible to blind players via screen reader (Tolk) and full keyboard navigation.
---

# BazaarAccess - Claude Guide

A BepInEx mod making "The Bazaar" accessible to blind players via screen reader (Tolk) and full keyboard navigation.

## Tech Stack
- **C# / .NET Framework 4.6** targeting Unity 2019.4.16
- **BepInEx 5.x** for mod loading
- **HarmonyLib** for runtime patching
- **TolkDotNet** for screen reader output (NVDA, JAWS, etc.)

## Project Structure
```
BazaarAccess/
├── Accessibility/    # Framework: AccessibilityMgr, BaseScreen, BaseUI, AccessibleMenu
├── Core/             # TolkWrapper, KeyboardNavigator, MessageBuffer, CoroutineHelper
├── Gameplay/         # Main gameplay logic (see sub-modules below)
│   ├── Combat/       # HealthTracker, CardStatsTracker, EffectFormatter
│   ├── Navigation/   # Sub-navigators (see below)
│   │   ├── NavigationTypes.cs      # Enums: NavigationSection, RecapSection, NavItemType, NavItem
│   │   ├── BoardStashNavigator.cs  # Board/stash refresh, slot nav, stash toggle, capacity
│   │   ├── SelectionNavigator.cs   # Selection data (shop/encounters/loot), card descriptions
│   │   ├── GameplayAnnouncer.cs    # Announcements, state descriptions, game actions (exit/reroll)
│   │   ├── HeroNavigator.cs        # Hero stats/skills navigation
│   │   ├── EnemyNavigator.cs       # Enemy board/stats/skills navigation
│   │   ├── RecapNavigator.cs       # Post-combat recap mode navigation
│   │   ├── DetailReader.cs         # Line-by-line detail reading
│   │   └── VisualSelector.cs       # Visual selection feedback
│   ├── GameplayScreen.cs       # Input router, state callbacks
│   ├── GameplayNavigator.cs    # Facade: coordinates all sub-navigators, section/item nav
│   ├── ActionMenuHandler.cs    # Action mode overlay (sell/upgrade/enchant/move/reorder)
│   ├── CombatInputHandler.cs   # Combat-mode input routing (B/G/V/F board navigation)
│   ├── ReplayInputHandler.cs   # Post-combat replay/recap input routing
│   ├── CardReading/          # Sub-modules for card data reading
│   │   ├── TextResolver.cs       # Localized text and token resolution
│   │   ├── CardProperties.cs     # Name, tier, size, price, tags, descriptions
│   │   ├── QuestReader.cs        # Quest conditions, progress, rewards
│   │   ├── DetailLineBuilder.cs  # Detail lines, short/detailed descriptions, stats
│   │   ├── EncounterReader.cs    # Encounter info, PvP opponent data
│   │   ├── PropertyDescriber.cs  # Tag and keyword descriptions (I key)
│   │   └── RankReader.cs         # Player rank, ranked mode detection
│   ├── ItemReader.cs           # Facade: delegates to CardReading/ sub-modules
│   ├── CombatDescriber.cs      # Combat narration (batched/individual modes)
│   ├── ActionHelper.cs         # Buy/sell/move/reorder commands
│   ├── PedestalManager.cs      # Pedestal detection, caching, upgrade/enchant actions
│   └── TierHelper.cs           # Tier progression utilities
├── Patches/          # Harmony patches + event handlers
│   ├── StateChangePatch.cs     # Core state transitions, event subscriptions, debounce
│   ├── CombatEventHandler.cs   # Combat lifecycle (start/end/result)
│   ├── CardEventHandler.cs     # Card transactions (buy/sell/equip/enchant/upgrade)
│   └── ErrorEventHandler.cs    # Error events (no space, can't afford, unsellable)
├── Screens/          # Main screens (MainMenu, HeroSelect, Collection, BattlePass, ChestScene)
├── UI/               # Dialog/popup UIs including Login/ subdirectory
└── Plugin.cs         # Entry point
```

## Core Architecture

### Focus Management (AccessibilityMgr.cs)
- **_currentScreen**: Active main screen (IAccessibleScreen)
- **_uiStack**: Stack of popup UIs (IAccessibleUI) - top gets input priority
- `SetScreen()` clears UI stack; `ShowUI()`/`HideUI()` push/pop dialogs

### Keyboard Input Flow (KeyboardNavigator.cs)
```
Unity OnGUI → MapKey(event) → AccessibleKey enum → AccessibilityMgr.HandleInput()
              → FocusedUI?.HandleInput() OR CurrentScreen?.HandleInput()
```

### Screen Reader Output (TolkWrapper.cs)
- `Speak(text, interrupt)` - 0.3s dedup window prevents spam
- `SpeakForced()` - bypasses dedup for intentional repeats

### Menu Pattern (AccessibleMenu.cs)
Composition-based navigation used by all screens/UIs:
- `AddOption(text, onConfirm, onRead?, onAdjust?)`
- Up/Down navigates
- Home/End/PageUp/PageDown for fast navigation

## Key Files for Common Tasks

| Task | Files |
|------|-------|
| Add new game screen | `Screens/`, implement `IAccessibleScreen`, register in `ViewControllerPatch.cs` |
| Add new popup/dialog | `UI/`, extend `BaseUI`, register in `PopupPatch.cs` |
| Hook game events | `Patches/StateChangePatch.cs` (subscribe), `CombatEventHandler.cs`, `CardEventHandler.cs`, `ErrorEventHandler.cs` |
| Modify item reading | `Gameplay/CardReading/` (sub-modules), `Gameplay/ItemReader.cs` (facade) |
| Combat narration | `Gameplay/CombatDescriber.cs`, `Combat/HealthTracker.cs`, `Combat/CardStatsTracker.cs`, `Combat/EffectFormatter.cs` |
| Keyboard shortcuts | `Core/KeyboardNavigator.cs` (mapping), `GameplayScreen.cs` (router), `CombatInputHandler.cs`, `ReplayInputHandler.cs` |
| Item actions | `Gameplay/ActionHelper.cs` (buy/sell/move/reorder), `PedestalManager.cs` (upgrade/enchant) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ali-Bueno/bazaar-access](https://github.com/Ali-Bueno/bazaar-access) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
