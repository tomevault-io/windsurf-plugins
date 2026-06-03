---
trigger: always_on
description: macOS app for browsing, editing, and managing **agent harness artifacts** on disk: skills (`SKILL.md`), MCP server configs, and plugins across Cursor, Claude Code, Codex, Hermes, Pi, and OpenCode. Includes a menu-bar agent monitor for live session status.
---

# AGENTS.md — Skillz

macOS app for browsing, editing, and managing **agent harness artifacts** on disk: skills (`SKILL.md`), MCP server configs, and plugins across Cursor, Claude Code, Codex, Hermes, Pi, and OpenCode. Includes a menu-bar agent monitor for live session status.

## Repo layout

```
skillz-macos/
├── CLAUDE.md / AGENTS.md     # This file (keep in sync)
├── README.md                 # Public status, agent detection, CI, Sparkle placeholders
├── .github/workflows/ci.yml  # Debug build + skillzTests on macos-26
└── skillz/                   # Xcode project root
    ├── skillz.xcodeproj
    ├── skillz/               # App target sources
    │   ├── skillzApp.swift   # @main — WindowGroup, MenuBarExtra, Settings
    │   ├── Views/            # SwiftUI UI (MainWindowView, lists, editor, sheets)
    │   ├── Services/         # Catalog discovery, file I/O, agent engines, hooks
    │   ├── Models/           # SkillItem, MCPItem, PluginItem, AgentSession, …
    │   ├── Settings/         # AppSettings, SettingsView tabs
    │   ├── Theme/            # Typography, colors, shared components
    │   ├── Notch/            # Dormant legacy NSPanel notch UI sources (not wired at runtime)
    │   ├── Resources/        # skillz-agent-notify.sh (bundled, installed to ~/.skillz/bin)
    │   ├── Assets.xcassets/  # AppIcon, Skillz* colors, PlatformIcon* SVG/vector assets
    │   └── ThirdParty/       # icon notices / third-party licenses
    ├── skillzTests/          # Swift Testing unit tests
    └── skillzUITests/        # UI tests (launch smoke)
```

Xcode uses **PBXFileSystemSynchronizedRootGroup** — new files under `skillz/skillz/` are picked up automatically; no manual `pbxproj` edits for most adds.

## Stack

| Layer | Choice |
|-------|--------|
| UI | SwiftUI + AppKit (`NSPanel`, `NSHostingView`, `MenuBarExtra`) |
| Language | Swift 5, `@MainActor` view models |
| Persistence | Direct file I/O (no Core Data); `~/Library/Application Support/Skillz/agent-state.json` for agent snapshots |
| Tests | Swift Testing (`@Test` in `skillzTests`) |
| Icons | Asset-catalog platform icons (`PlatformIcon*`) rendered as template images in sidebar/menu-bar surfaces |

**Deployment:** macOS **26.2+**, bundle ID `robertcourson.skillz`, **not sandboxed** (`skillz.entitlements` → `com.apple.security.app-sandbox` = false) so the app can read `~/.cursor`, `~/.claude`, `~/.codex`, etc. UI product name is **Skills** (`AppBrand.name`); current marketing version **1.0.2**.

## Build and test

From `skillz/` (directory containing `skillz.xcodeproj`):

```bash
xcodebuild -scheme skillz -destination 'platform=macOS' build
xcodebuild -scheme skillz -destination 'platform=macOS' test
```

- **Scheme:** `skillz`
- **Unit tests:** `skillzTests` — 48 `@Test` functions (frontmatter, catalog filter, platform paths, source detection, bare-home-not-detected, agent engine, session-adapter liveness/id stability, process runner, hooks, startup hook policy, file service, legacy notch layout/view-model, process exclusions, session dedup, discovery smoke)
- **UI tests:** `skillzUITests` — launch/performance (slow); skip unless needed
- **CI:** GitHub Actions (`.github/workflows/ci.yml`) — Debug build + `-only-testing:skillzTests` on `macos-26`

Release checklist is inline in `skillz.entitlements` (Developer ID, archive, notarize, staple). Public builds ship as GitHub Release DMGs (`Skills-macOS-v*.dmg`); update/signing/Sparkle details live in `README.md` and `docs/UPDATES.md`.

## Architecture

### App entry and scenes

- **`skillzApp`**: `WindowGroup` → `MainWindowView` with hidden titlebar; `MenuBarExtra` → agent menu + **menu-bar glyph** (`SkillzMenuBarIconView` — template `sparkles` SF Symbol the system tints, so it stays visible on light/dark menu bars; the app icon is a near-black mark unusable here); `Settings` scene.
- **`SkillzStartupConfigurator`**: on first appear, starts `AgentSessionStore`, defers initial hook install/repair until onboarding is complete, reopens watching on app activation, and stops monitoring on termination.
- **`SkillzWindowChromeCleaner`**: AppKit bridge — clears native toolbar/titlebar chrome and hides stray AppKit sidebar-toggle buttons in the title bar.
- **`OnboardingView`**: first-launch sheet (`settings.hasCompletedOnboarding`); shows live source/tool detection for all tracked platforms and toggles menu-bar waiting count, inspector, and automatic hook repair before catalog use.

### Main window (`NavigationSplitView`)

| Column | View | Role |
|--------|------|------|
| Sidebar | `SidebarView` | Library sections (All / Skills / MCPs / Plugins) + platform filters |
| Content | `ItemListView` | Searchable catalog list (`SkillzListRow`) |
| Detail | `DetailContainerView` | Skill editor, MCP/plugin detail; optional `InspectorView` |

**State:** `CatalogStore` (snapshot, filters, selection, FSEvents rescan), `EditorDocument` (markdown autosave), `AppSettings` (`@AppStorage`).

**Top bar** (full-width row above the split view, `.zIndex(1)` so its hairline masks inset-sidebar shadow bleed):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robzilla1738/skillz-macos](https://github.com/robzilla1738/skillz-macos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
