---
trigger: always_on
description: devecocli build --product default --config-name debug
---

# AGENTS.md

## Build & Run

```bash
devecocli build --product default --config-name debug
devecocli build --product default --config-name release
devecocli run --product default --config-name debug
```

Two products: **`default` (HarmonyOS)** and **`ohos` (OpenHarmony)**. Only one module: **`home`** (entry HAP).

## Single-Page Architecture

There is ONE page: **`pages/Index.ets`** (~790 lines). No router-based navigation. All "navigation" is via:
- Dialog overlays — settings, bookmarks, history, downloads etc. are `@CustomDialog` structs.
- Tab switching — `meowTabsBunch` in `hosts/app/tabs/`.
- Sidebar panel switching — `components/parts/Sidebar/`.

Do NOT add new router pages. Add dialogs instead.

## The CatsTypicalFramework

`catstypicalframework/` is the custom micro-framework for lifecycle, state wiring, and utilities:

| Directory | Role |
|---|---|
| `catstypicalframework/flows.ets` | **Orchestrator.** All lifecycle callbacks from `EntryAbility` delegate here. |
| `catstypicalframework/hosts/` | Framework-level hosts: `meowLayout`, `meowAvoid`, `meowMisc` |
| `catstypicalframework/utils/` | 19 utilities: logging, storage, URL parsing, clipboard, colors, etc. |
| `hosts/app/` | Feature hosts: tabs, downloads, scratchboard |
| `hosts/system/` | System hosts: `meowAppHost` (singleton), `meowUiHost` (per-window), messaging |
| `hosts/userdata/` | User data: `meowManagedSettings`, `meowBookmarkStash`, `meowHistorian` |

## State Management: @ObservedV2 + AppStorageV2

**All hosts are `@ObservedV2` classes with `@Trace` on every reactive property.**

### Wiring Pattern

```typescript
// In a @ComponentV2 struct:
@Local myApp: meowAppHost = getCreateMyApp();
@Local myUi: meowUiHost = AppStorageV2.connect(meowUiHost, `meowUiHost_${this.myWindowId}`)!;
```

`getCreateMyApp()` returns the singleton `meowAppHost`. `AppStorageV2.connect()` retrieves a typed per-window instance.

### Global vs. Per-Window State

**Global hosts** (shared, instantiated ONCE, no key suffix):
`meowBookmarkStash`, `meowHistorian`, `meowKeyboardMan`, `meowManagedSettings`, `meowBoardHolder`, `meowDawnloada`, `meowResources`, `meowAppHost`

**Per-window hosts** (keyed `meowXxx_${windowId}`):
`meowUiHost`, `meowDialogManager`, `meowMisc`, `meowAvoid`, `meowLayout`, `meowTabsBunch`

When adding a new host, register it in **both**:
1. `earlyOnCreate()` in `catstypicalframework/flows.ets` (creation)
2. `reportWindowClose()` in `meowUiHost` (`AppStorageV2.remove()` cleanup)

### Messaging System

`@ObservedV2` instances as transient message buses:
- **Cross-window**: set `getCreateMyApp().crossAppMessageTunnel` → all windows via `@Monitor('myApp.crossAppMessageTunnel')`
- **Local**: set `myUi.localMessageTunnel` → same window via `@Monitor('myUi.localMessageTunnel')`

## Settings: Hard-Coded Integer Indices

~100+ settings stored in `meowManagedSettings.settingsData[]`, accessed by **hard-coded integer indices** (no hash lookup for performance). No central index registry.

UI getters in **`components/settings.ets`** (e.g., `settingsData[7]` → `useEffects()`, `settingsData[1]` → `titleBarPositionBottom()`).

When adding a setting:
1. Append to the `defaultSettingsEntries` array.
2. Add a getter in `components/settings.ets`.
3. Use `setByIndex()` which validates types.

**Never reorder existing entries** — that breaks persisted user settings.

## Dialog Architecture

All dialogs follow this pattern:

```typescript
@CustomDialog
struct WoofSomeDialog {
  controller: CustomDialogController;
  @Require myUi: meowUiHost;

  build() {
    WoofUniFrame({ controller: this.controller, myUi: this.myUi, title: '...' }) {
      // Dialog content
    }
  }
}
```

- **`WoofUniFrame`** (`dialogs/WoofUniFrame.ets`) — mandatory outer shell: draggable title bar, tutorial, confirm/cancel, auto-registration with `meowDialogManager`.
- **`MeowPlaceForDialogs`** (`components/parts/MeowPlaceForDialogs.ets`) — holds ~30 `CustomDialogController` fields. Dialogs are requested by writing to `myUi.myDialog.lastRequest` (triggers `@Monitor`).
- Dialogs register/unregister with `meowDialogManager` in `aboutToAppear()`/`aboutToDisappear()`. `goBackUni()` checks `this.myDialog.isEmpty()` first.

## Multi-Window Lifecycle

- `earlyOnCreate()` runs for **every** window.
- Shared-host init blocks in `earlyOnWindowStageCreate()` run ONLY for the **first** window.
- On close, `reportWindowClose()` cleans up per-window `AppStorageV2` keys.
- New windows: `EntryAbilityStage.onAcceptWant()` → `generateTimeBasedWindowId()`.

## Naming Conventions

| Prefix | Used For | Examples |
|---|---|---|
| `meow` | Host/model classes | `meowAppHost`, `meowUiHost`, `meowTabsBunch` |
| `Moo` | Common UI components | `MooSymbolButton`, `MooProgress`, `MooTabSegmentV2` |
| `Woof` | Dialogs | `WoofSettings`, `WoofBookmarks`, `WoofUniFrame` |
| `Meow` | Feature-level components | `MeowAddressBar`, `MeowSidebar`, `MeowPlaceForDialogs` |

## Performance Patterns

- **`animateToImmediately(defaultAnimation(), () => { ... })`** — use this (NOT `animateTo()`) for state-driven layout. `defaultAnimation()` reads spring-motion from user settings.
- **Lazy imports** (`import lazy { ... }`) — used extensively for code splitting. Do NOT remove `lazy`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awaLiny2333/LinysBrowser_NEXT](https://github.com/awaLiny2333/LinysBrowser_NEXT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
