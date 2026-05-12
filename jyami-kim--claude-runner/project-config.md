---
trigger: always_on
description: macOS menu bar app that monitors active Claude Code sessions via hook scripts. Displays a traffic-light style status icon and a popover listing all sessions with their state, app icon, elapsed time, and activity text. Clicking a session row focuses the corresponding terminal/IDE window.
---

# claude-runner

macOS menu bar app that monitors active Claude Code sessions via hook scripts. Displays a traffic-light style status icon and a popover listing all sessions with their state, app icon, elapsed time, and activity text. Clicking a session row focuses the corresponding terminal/IDE window.

## Tech Stack

- **Language:** Swift 5.9
- **UI:** SwiftUI (panel popover) + AppKit (menu bar, NSImage rendering)
- **Build:** Swift Package Manager (`Package.swift`)
- **Platform:** macOS 13+

## Project Structure

```
claude-runner/
├── Entry/                          # Executable target entry point
├── Sources/                        # ClaudeRunnerLib target
│   ├── App/
│   │   ├── ClaudeRunnerApp.swift   # App bootstrap
│   │   └── AppDelegate.swift       # NSStatusItem, PopoverPanel (NSPanel), watcher setup
│   ├── Models/
│   │   ├── SessionState.swift      # SessionState enum, SessionEntry, StateCounts, StateStore
│   │   ├── AppSettings.swift       # @AppStorage settings, IconStyle, SessionDisplayFormat, AppLanguage enums
│   │   └── Strings.swift           # Centralized UI strings (Korean + English)
│   ├── Views/
│   │   ├── SessionListView.swift   # Panel content: session list + session row + Revive button
│   │   ├── SettingsView.swift      # Settings window UI (6 sections)
│   │   └── StatusIcon.swift        # Menu bar icon updater
│   ├── Services/
│   │   ├── HookInstaller.swift     # Installs hook script to Application Support
│   │   ├── HookRegistrar.swift     # Registers hooks in ~/.claude/settings.json (no jq needed)
│   │   ├── SessionDirectoryWatcher.swift  # kqueue-based directory watcher
│   │   ├── SessionScanner.swift    # Scans running processes for orphaned Claude sessions
│   │   ├── LoginItemManager.swift  # SMAppService wrapper for launch-at-login
│   │   ├── NotificationService.swift  # UNUserNotificationCenter wrapper + click-to-focus
│   │   ├── UpdateChecker.swift     # GitHub Releases API update checker
│   │   ├── TerminalFocuser.swift   # Protocol + dispatcher + shared helpers
│   │   └── Focusers/
│   │       ├── ITermFocuser.swift         # iTerm2 AppleScript TTY matching
│   │       ├── TerminalAppFocuser.swift   # Terminal.app AppleScript TTY matching
│   │       ├── JetBrainsFocuser.swift     # Toolbox CLI + worktree resolution
│   │       ├── WarpFocuser.swift          # Warp System Events window title matching
│   │       └── DefaultFocuser.swift       # NSRunningApplication fallback
│   └── Extensions/
│       ├── BundleIdentifier+AppInfo.swift  # Bundle ID → app name/icon resolver
│       ├── DesignTokens.swift      # Colors, dimensions, spacing constants
│       └── NSImage+TrafficLight.swift  # Menu bar icon renderers (4 styles)
├── Scripts/
│   └── claude-runner-hook.sh       # Claude Code hook → writes session JSON (tmux-aware)
├── release.sh                      # Release script: version bump → tag → push (triggers CI)
├── Resources/
│   ├── AppIcon.icns / .svg
│   └── Info.plist
├── Tests/                          # ClaudeRunnerTests target
│   ├── SessionStateTests.swift
│   ├── StateStoreTests.swift
│   ├── HookStateTransitionTests.swift
│   ├── DesignTokensTests.swift
│   ├── TrafficLightTests.swift
│   ├── AppSettingsTests.swift
│   ├── LoginItemManagerTests.swift
│   ├── NotificationServiceTests.swift
│   ├── HookRegistrarTests.swift
│   ├── AppInfoTests.swift
│   ├── WorktreeResolutionTests.swift
│   ├── SessionScannerTests.swift
│   └── UpdateCheckerTests.swift
└── Package.swift
```

## Architecture

1. **Hook script** (`claude-runner-hook.sh`) receives Claude Code hook events via stdin JSON, writes per-session `.json` files to `~/Library/Application Support/claude-runner/sessions/`. Validates the caller is Claude Code by checking the PPID chain for the `claude` binary (prevents other tools like opencode from creating sessions). Captures `terminal_bundle_id` and `tty` from the parent process chain. Supports tmux environments via `#{client_pid}` and `#{client_tty}` fallbacks. Captures `last_message` (from Stop events) and `current_activity` (from tool use events).
2. **HookInstaller** copies the hook script from the .app bundle (`Contents/Resources/`) to `~/Library/Application Support/claude-runner/hooks/` on every launch.
3. **HookRegistrar** idempotently registers hooks in `~/.claude/settings.json` using `JSONSerialization` (no jq dependency). Adds 8 hook events + 3 notification matchers.
4. **SessionDirectoryWatcher** monitors the sessions directory via kqueue (DispatchSource) and triggers `StateStore.reload()`.
5. **StateStore** reads JSON files, prunes stale sessions, publishes `sessions` + `counts`, and supports `reviveSessions()` for orphaned process recovery and dead session cleanup.
6. **StatusIcon** renders the menu bar icon using `NSImage.icon(style:counts:)`.
7. **PopoverPanel** (NSPanel subclass) displays the session list. Uses `.regularMaterial` background with rounded corners.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jyami-kim/claude-runner](https://github.com/jyami-kim/claude-runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
