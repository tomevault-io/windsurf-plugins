---
trigger: always_on
description: This file is for AI coding agents only.
---

# Quick Search - AI Agent Guide

This file is for AI coding agents only.
Use it as the primary implementation playbook for building new features and updating existing ones in this repository.

## 1) Project Snapshot

- App type: Android launcher/search app
- Language: Kotlin
- UI: Jetpack Compose + Material 3
- Architecture: MVVM with unidirectional data flow
- Package: `com.tk.quicksearch`
- Core behavior: unified search for apps, app shortcuts, contacts, calendar events, files, device settings, app settings, web, and tools (calculator/unit/date/ai search)

## 2) Core Architecture (Do Not Bypass)

### Layers

- **UI layer**: composables under `search/`, `settings/`, `widgets/`, `overlay/`, `onboarding/`
- **ViewModel layer**: orchestration and state in `search/core/`
- **Data layer**: repositories and preferences in `search/data/`

### Single source of truth

- Main state: `SearchUiState` in `search/core/SearchModels.kt`
- Additional state models: `search/core/SearchStateModels.kt`
- Main orchestrator: `search/core/SearchViewModel.kt`
- State transport: `StateFlow`
- Update pattern: always use immutable `copy(...)` through ViewModel update helpers

### State conventions

- Use sealed classes for visibility/loading/no-results/showing-results states.
- Do not perform permission checks directly inside UI composables when state already exists in `SearchUiState`.
- Prefer adding state in `SearchModels.kt`/`SearchStateModels.kt` before wiring UI logic.

## 3) Repository and Preference Patterns

### Repositories (search data)

- `AppsRepository.kt`
- `ContactRepository.kt`
- `FileSearchRepository.kt`
- `CalendarRepository.kt`
- `search/data/appShortcutRepository/AppShortcutRepository.kt`
- `search/deviceSettings/DeviceSettingsRepository.kt`

### Preferences

- Central facade: `search/data/userAppPreferences/UserAppPreferences.kt`
- Startup facade: `search/data/userAppPreferences/StartupPreferencesFacade.kt`
- Specialized modules: `search/data/preferences/`
- Keep new preferences modular; delegate through `UserAppPreferences`.
- `SearchHistoryPreferences.kt` stays in `search/searchHistory/`.

### Cache/Startup behavior

- App cache: `search/data/AppCache.kt`
- Startup strategy is phased (critical prefs -> remaining prefs -> deferred init) via `app/startup/StartupCoordinator.kt` and `search/core/SearchStartupCoordinator.kt`.
- Avoid expensive synchronous work on startup paths.

## 4) Key Feature Areas and Main Files

### Search core

- `search/core/SearchViewModel.kt`
- `search/core/SearchViewModelSearchOperations.kt`
- `search/core/UnifiedSearchHandler.kt`
- `search/core/SectionManager.kt`
- `search/core/SearchSectionRegistry.kt`
- `search/core/SearchModels.kt`
- `search/core/SearchStateModels.kt`

### Search UI composition

- `search/searchScreen/SearchScreen.kt`
- `search/searchScreen/SearchScreenContent.kt`
- `search/searchScreen/SectionRenderingComposables.kt`
- `search/searchScreen/SearchScreenLayout.kt`
- `search/searchScreen/searchScreen/SearchRoute.kt`
- `search/searchScreen/searchScreenLayout/`

### Feature packages

- Apps: `search/apps/`
- App settings results: `search/appSettings/`
- App shortcuts: `search/appShortcuts/`
- Contacts: `search/contacts/`
- Calendar: `search/calendar/`
- Files: `search/files/`
- Device settings: `search/deviceSettings/`
- Web suggestions: `search/webSuggestions/`
- Search history: `search/searchHistory/`
- Fuzzy search: `search/fuzzy/`

### Search engines and tools

- Search engines: `searchEngines/`
- Orchestration/debounce: `searchEngines/SecondarySearchOrchestrator.kt`
- AI Search: `tools/aiSearch/`
- Built-in tools: `tools/calculator/`, `tools/unitConverter/`, `tools/dateCalculator/`, `tools/aiTools/`

### Overlay mode

- `overlay/OverlayActivity.kt`
- `overlay/OverlayRoot.kt`
- `overlay/OverlayModeController.kt`
- App theme color utils: `search/searchScreen/AppThemeUtils.kt`

### Settings

- Root: `settings/SettingsScreen.kt`
- Detail navigation: `settings/navigation/`
- Detail sections: `settings/settingsDetailScreen/`
- Appearance: `settings/appearanceSettings/`
- Search engine settings: `settings/searchEngineSettings/`
- App shortcuts settings: `settings/appShortcutsSettings/`
- Shared settings route/state: `settings/shared/settingsRoute/`

## 5) UI Design System Rules

### Tokens and colors

- Use `shared/ui/theme/DesignTokens.kt` for spacing, shapes, dimensions.
- Use `shared/ui/theme/AppColors.kt` for colors and wallpaper-aware surfaces.
- Avoid hardcoded dp, corner radii, and color literals unless extending tokens.

### Shared components first

- Prefer components in `shared/ui/components/` before creating new one-off UI.
- Keep composables stateless where possible; hoist state to ViewModel/parent.
- Keep `modifier: Modifier = Modifier` in reusable composables.
- Try existing design tokens first.
- Add new design tokens when they are reusable across multiple places.
- If a value is file-specific, add a top-level `const` in that file instead of a global token.

### Responsive behavior

- Device/orientation helpers: `shared/util/DeviceUtils.kt`
- Preserve tablet behavior for app grid and search engine layout.

## 6) Search and Ranking Behavior

### Ranking stack

- Traditional ranking: `search/common/SearchRankingUtils.kt`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teja2495/quick-search](https://github.com/teja2495/quick-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
