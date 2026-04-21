---
trigger: always_on
description: cctop is a macOS menubar app for monitoring AI coding sessions across workspaces. It tracks session status (idle, working, needs attention) via tool-specific plugins and allows jumping to sessions. Works with Claude Code and opencode. Also includes a Raycast extension that reads the same session data.
---

# CLAUDE.md - Development Guide for cctop

## Project Overview

cctop is a macOS menubar app for monitoring AI coding sessions across workspaces. It tracks session status (idle, working, needs attention) via tool-specific plugins and allows jumping to sessions. Works with Claude Code and opencode. Also includes a Raycast extension that reads the same session data.

## MUST FOLLOW: Development Principles

- Do NOT modify the user's tool configuration without explicit consent (e.g. installing plugins, editing hooks or settings files)
- Do NOT make breaking changes that require users to restart running sessions to reconnect to the app
  - Newly added features being unavailable until restart is acceptable

## Architecture

```
cctop/
├── menubar/           # Swift/SwiftUI app (menubar + hook CLI)
│   ├── CctopMenubar.xcodeproj/
│   ├── CctopMenubar/
│   │   ├── CctopApp.swift         # App entry point
│   │   ├── AppDelegate.swift      # NSStatusItem + FloatingPanel toggle
│   │   ├── FloatingPanel.swift    # NSPanel subclass (stays open)
│   │   ├── Models/                # Session, SessionStatus, HookEvent, Config (shared)
│   │   ├── Views/                 # PopupView, SessionCardView, QuitButton, etc.
│   │   ├── Services/              # SessionManager, FocusTerminal
│   │   └── Hook/                  # cctop-hook CLI target only
│   │       ├── HookMain.swift     # CLI entry point (stdin, args, dispatch)
│   │       ├── HookInput.swift    # Codable struct for Claude Code hook JSON
│   │       ├── HookHandler.swift       # Core logic (transitions, cleanup, PID)
│   │       ├── SessionNameLookup.swift # Session name from transcript/index
│   │       └── HookLogger.swift        # Per-session logging
│   └── CctopMenubarTests/
├── raycast/           # Raycast extension (TypeScript/React)
│   ├── package.json           # Extension manifest (single command)
│   ├── src/
│   │   ├── show-sessions.tsx  # Main list command
│   │   ├── sessions.ts        # Session loading, parsing, grouping
│   │   ├── actions.ts         # Jump-to-session logic
│   │   ├── status-ui.ts       # Status color/label/icon mapping
│   │   └── types.ts           # TypeScript interfaces
│   ├── metadata/              # Store screenshots
│   ├── CHANGELOG.md           # Store changelog
│   └── README.md              # Store README
├── plugins/cctop/     # Claude Code plugin
│   ├── .claude-plugin/plugin.json
│   ├── hooks/hooks.json
│   └── skills/cctop-setup/SKILL.md
├── plugins/opencode/  # opencode plugin (JS, translates events to cctop-hook calls)
│   ├── plugin.js      # Event handler, calls cctop-hook binary
│   └── package.json   # Plugin manifest
├── plugins/pi/        # pi coding agent extension (TS, translates events to cctop-hook calls)
│   └── cctop.ts       # Extension entry point, calls cctop-hook binary
├── scripts/
│   ├── bundle-macos.sh        # Build and bundle .app
│   ├── sign-and-notarize.sh   # Code sign + Apple notarization
│   ├── generate-appcast.sh    # Sparkle appcast (multi-arch)
│   └── bump-version.sh        # Version bumper (all files)
├── packaging/
│   └── homebrew-cask.rb  # Homebrew cask template
└── .claude-plugin/
    └── marketplace.json  # For local plugin installation
```

### Swift Menubar App

The macOS menubar app is built with Swift/SwiftUI. It uses a custom `AppDelegate` with `NSStatusItem` and a `FloatingPanel` (NSPanel subclass) that stays open until the user clicks the menubar icon again.

**Location:** `menubar/`

**Build:**
```bash
# Build from command line
xcodebuild build -project menubar/CctopMenubar.xcodeproj -scheme CctopMenubar -configuration Debug -derivedDataPath menubar/build/ CODE_SIGN_IDENTITY="-"

# Run the app
open menubar/build/Build/Products/Debug/CctopMenubar.app

# Run tests
xcodebuild test -project menubar/CctopMenubar.xcodeproj -scheme CctopMenubar -configuration Debug -derivedDataPath menubar/build/
```

**Visual verification:** Open the Xcode project and use SwiftUI Previews (Canvas) for instant visual feedback. All views have `#Preview` blocks with mock data.

**Data flow:** The menubar app reads `~/.cctop/sessions/*.json` files. These are written by `cctop-hook` (Swift CLI), which is called by all plugins (Claude Code hooks, opencode JS plugin, pi TS extension). Both Xcode targets share model code.

**Key files:**
- `menubar/CctopMenubar/AppDelegate.swift` — NSStatusItem + FloatingPanel management
- `menubar/CctopMenubar/FloatingPanel.swift` — NSPanel subclass (persistent popup)
- `menubar/CctopMenubar/Views/PopupView.swift` — Main popup layout
- `menubar/CctopMenubar/Views/SessionCardView.swift` — Session card component
- `menubar/CctopMenubar/Models/Session.swift` — Session data model (Codable, shared)
- `menubar/CctopMenubar/Models/HookEvent.swift` — Hook event enum + transition logic (shared)
- `menubar/CctopMenubar/Models/Config.swift` — JSON config, sessions dir (shared)
- `menubar/CctopMenubar/Services/SessionManager.swift` — File watching + session loading

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [st0012/cctop](https://github.com/st0012/cctop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
