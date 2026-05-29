---
trigger: always_on
description: Instructions for AI coding agents working with this codebase.
---

# AGENTS.md

Instructions for AI coding agents working with this codebase.

<!-- opensrc:start -->

## Source Code Reference

Source code for dependencies is available in `opensrc/` for deeper understanding of implementation details.

See `opensrc/sources.json` for the list of available packages and their versions.

Use this source code when you need to understand how a package works internally, not just its types/interface.

### Fetching Additional Source Code

To fetch source code for a package or repository you need to understand, run:

```bash
npx opensrc <package>           # npm package (e.g., npx opensrc zod)
npx opensrc pypi:<package>      # Python package (e.g., npx opensrc pypi:requests)
npx opensrc crates:<package>    # Rust crate (e.g., npx opensrc crates:serde)
npx opensrc <owner>/<repo>      # GitHub repo (e.g., npx opensrc vercel/ai)
```

<!-- opensrc:end -->

## Agent skills

### Issue tracker

Issues are tracked as local markdown files under `.local/<feature>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Default vocabulary — needs-triage, needs-info, ready-for-agent, ready-for-human, wontfix. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context — CONTEXT.md at the repo root. See `docs/agents/domain.md`.

---

## PROJECT: Scrolodex

macOS menu-bar app for switching between windows by scrolling. Swift 6.1 with SPM, targeting macOS 14+. Zero external dependencies.

## STRUCTURE

```
Scrolodex/
├── Sources/
│   ├── ScrolodexCore/             # Core domain library (36 files, 6 subdirs + 3 root)
│   │   ├── EventRouter/            # Stateless classifier + session state + action enums
│   │   ├── Navigation/             # Coordinator, session, protocols, release behavior
│   │   ├── Triggers/               # 4 trigger model types + config + dock protocol
│   │   ├── Overlay/                # Theme, placement, presentation modes, peek, display config
│   │   ├── WindowQuery/            # Candidate model, CG parser, AX accessor, stack filter/query
│   │   ├── Desktop/                # SpaceSwitcher (SkyLight private APIs)
│   │   ├── SettingDefaults.swift   # Canonical fallback values for all settings
│   │   ├── Log.swift               # NSLog wrapper with debug toggle
│   │   └── MenuBarIconConfiguration.swift
│   └── Scrolodex/                 # macOS app layer (30 files)
│       ├── App/                    # main.swift, AppDelegate, StatusBarController, PermissionController
│       ├── Input/                  # EventTapController, TrackpadGestureObserver, DockObserver, DockActionHandler
│       ├── Overlay/                # AppKit views: OverlayController, OverlayView, TileOverlayView, Peek views
│       ├── Settings/               # SwiftUI settings: TriggerSettingsStore (@Observable), SettingsView, GlobalVisualSettings
│       ├── WindowManagement/       # CGWindowStackProvider, AccessibilityWindowController, ThumbnailProvider, DockGeometry
│       └── Resources/              # MenuBarIcon.svg
└── Tests/
    └── ScrolodexCoreTests/        # Swift Testing (24 files, mirrors core subdirs)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Domain language | `CONTEXT.md` | Single source of truth — read first |
| Adding a trigger type | `ScrolodexCore/Triggers/TriggerModels.swift` + `EventRouter/EventClassifier.swift` | Add enum case, config, routing |
| Modifying input handling | `Scrolodex/Input/` | EventTapController (CG tap), TrackpadGestureObserver (HID tap), DockObserver (AX) |
| Changing overlay UI | `Scrolodex/Overlay/` | AppKit views: Tooltip/List/Tile + Peek + animation |
| Navigation logic | `ScrolodexCore/Navigation/` | Coordinator → Session → ReleaseBehavior |
| Dock action handling | `ScrolodexCore/Triggers/DockActionHandling.swift` + `Scrolodex/Input/DockActionHandler.swift` | Protocol + shared handler |
| RouterAction domain types | `ScrolodexCore/EventRouter/RouterTypes.swift` | WindowAction, DesktopAction, DockAction, SystemAction |
| Window stack filtering | `ScrolodexCore/WindowQuery/WindowStackFilter.swift` | Scope, filter, monitor filters |
| Space switching | `ScrolodexCore/Desktop/SpaceSwitcher.swift` | SkyLight private APIs |
| Settings defaults | `ScrolodexCore/SettingDefaults.swift` | All canonical fallback values |
| Settings UI | `Scrolodex/Settings/` | SwiftUI: TriggerSettingsStore (@Observable), GlobalVisualSettings |
| Tests | `Tests/ScrolodexCoreTests/` | Swift Testing framework — mirrors core subdirs |
| Plans & issues | `.local/` | Markdown-based issue tracking |

## KEY ARCHITECTURE

- **Library + executable**: `ScrolodexCore` (pure domain, no AppKit, no persistence) → `Scrolodex` (macOS app, bridges to system APIs)
- **Protocol seams**: `WindowStackProviding`, `OverlayPresenting`, `WindowRaising` in `NavigationProtocols.swift` — the only integration points between core and app
- **Event Router pipeline**: raw CG events → `EventClassifier` (stateless, `Sendable` struct) → `RouterAction` → `NavigationCoordinator` (`@MainActor`) → `NavigationSession` (value type)
- **Mutable session state**: `RouterSessionState` (value type, `inout` parameter) owned by `EventTapController` — not inside the classifier

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaydenfyi/scrolodex](https://github.com/jaydenfyi/scrolodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
