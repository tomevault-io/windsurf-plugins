---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Breaks is a macOS menu-bar Pomodoro app written in SwiftUI. The codebase is organized into focused files across `Models/`, `Views/`, `Style/`, and `System/` directories. There is no Swift Package Manager, no test target, no lint config.

- Bundle ID: `com.gjinprelvukaj.Breaks`
- Deployment target: macOS 13.0
- Swift version: 5.0
- Sandboxed (`Breaks/Breaks.entitlements`): `com.apple.security.app-sandbox` + `com.apple.security.files.user-selected.read-only`

## Build / run

Open in Xcode and Cmd+R, or from CLI:

```sh
xcodebuild -project Breaks.xcodeproj -scheme Breaks -configuration Debug build
xcodebuild -project Breaks.xcodeproj -scheme Breaks clean
open Breaks.xcodeproj
```

To wipe persisted state during development (settings, timer state, session history, focus journal — all stored in `UserDefaults`):

```sh
defaults delete com.gjinprelvukaj.Breaks
```

There are no automated tests. UI is exercised manually via the menu-bar popover.

## Architecture

The codebase is organized into four directories under `Breaks/`:

### File structure

```
Breaks/
├── BreaksApp.swift              — @main entry point, MenuBarLabel, BreaksAppDelegate
├── SharedStorage.swift          — UserDefaults keys, TimerSnapshot for widget
├── Models/
│   ├── BreakTimer.swift         — Core timer controller (TickClock, IdlePrompt, mode cycling)
│   ├── TimerSettings.swift      — All user-tunable settings + DurationPreset
│   ├── SessionHistory.swift     — Daily completed-Pomodoro counts, streak calculation
│   └── FocusJournal.swift       — Today's focus, per-block labels, FocusBlockLog entries
├── Views/
│   ├── TimerPopover.swift       — Popover root, routes between onboarding/stats/settings/timer
│   ├── TimerContentView.swift   — Main timer page, MorningCheckIn, TodayDashboard, panels
│   ├── TimerRing.swift          — Circular progress ring, BreakSuggestionView
│   ├── OnboardingView.swift     — Three-step onboarding, PermissionRow
│   ├── StatsView.swift          — Statistics, StreakHeroCard, StreakHeatmap, WeeklyReview
│   ├── SettingsPanel.swift      — All settings sections, CollapsibleSection
│   └── ReusableComponents.swift — ModeSegmentedPicker, FooterLink, MinuteRow, etc.
├── Style/
│   ├── ButtonStyles.swift       — PressableButtonStyle, HapticPrimaryButtonStyle, etc.
│   ├── ColorExtensions.swift    — Color(hex:), toHex(), nilIfBlank
│   └── LiquidGlassBackgrounds.swift — .glassCard() modifier
└── System/
    ├── Hotkeys.swift            — HotkeyManager (Carbon RegisterEventHotKey)
    ├── LoginItemController.swift — SMAppService wrapper
    └── NotificationPermissions.swift — UNUserNotificationCenter authorization
```

### Entry point and shell
- `BreaksApp` (`@main`) declares a `MenuBarExtra` with `.menuBarExtraStyle(.window)` so the popover is a SwiftUI view, not an `NSMenu`.
- `MenuBarLabel` renders the status-bar icon + countdown. It observes `BreakTimer` for the icon and a separate `TickClock` for the remaining seconds (see "Tick decoupling" below).
- `BreaksAppDelegate` owns reopen handling and is the `UNUserNotificationCenterDelegate`.

### Core domain object: `BreakTimer`
`BreakTimer` (`Models/BreakTimer.swift`) is the main controller. It owns:
- `settings: TimerSettings` — all user-tunable values, persisted to `UserDefaults` per `@Published` property via `didSet`.
- `history: SessionHistory` — daily completed-Pomodoro counts, decay-aware streak calculation with per-ISO-week pause-day budget (`streakSnapshot(pauseDayBudget:)`).
- `journal: FocusJournal` — today's primary focus, per-block labels, `FocusBlockLog` entries with `FocusOutcome` (good/messy/skipped), weekly aggregations.
- `clock: TickClock` — see below.

Modes are `Mode.work | .shortBreak | .longBreak`; the cycle advances based on `completedWorkSessions` and `settings.sessionsPerCycle`.

### Tick decoupling (important when editing timer UI)
`BreakTimer.remaining` is a `@Published` property that **only updates at action boundaries** (start/pause/reset/setMode/fire). Per-second tick updates go to `clock.remaining` (a separate `ObservableObject`). This is intentional: it prevents the entire `TimerPopover` view tree from re-rendering every second. When adding UI that needs the live countdown, observe `TickClock`, not `BreakTimer`. The `didSet` on `remaining` mirrors action-boundary changes into `clock.remaining` so the two stay in sync.

### Persistence
All state lives in `UserDefaults.standard`. Three independently-loaded blobs:
- Scalar settings — keyed individually (`workMinutes`, `soundName`, hotkey codes, etc.).
- Timer state — `timerMode`, `timerRemaining`, `timerIsRunning`, `timerEndDate`, `completedWorkSessions`. On launch, if `isRunning` was true and `endDate` is still in the future, the timer auto-resumes; otherwise it restores remaining seconds or resets.
- JSON-encoded — `sessionHistory` (`[SessionRecord]`, pruned to 120 days) and `focusJournalState` (`FocusJournalState`, logs trimmed to 30 days).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GjinPrelvukaj/Breaks](https://github.com/GjinPrelvukaj/Breaks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
