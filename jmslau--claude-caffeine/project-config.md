---
trigger: always_on
description: ClaudeCaffeine is a macOS menu bar app (Swift 6.2, Swift Package Manager) that prevents your Mac from sleeping while Claude Code is actively working. It detects Claude Code activity through process inspection and file monitoring, holds IOKit sleep assertions, and optionally prevents lid-close sleep via a privileged helper.
---

# AGENTS.md

## Project Overview

ClaudeCaffeine is a macOS menu bar app (Swift 6.2, Swift Package Manager) that prevents your Mac from sleeping while Claude Code is actively working. It detects Claude Code activity through process inspection and file monitoring, holds IOKit sleep assertions, and optionally prevents lid-close sleep via a privileged helper.

**Target platform:** macOS 13+ (Ventura and later)
**Runtime:** Runs as a menu bar accessory app (`NSApplication.shared` with `.accessory` activation policy)

## Architecture

```
ClaudeCaffeine.swift          Entry point + AppDelegate (menu bar UI, poll loop, state machine)
    |
    +-- ClaudeHookMonitor          Scans ~/.claude/caffeine_sessions/ for active session files
    |
    +-- SleepAssertionManager      Holds/releases IOKit power assertions (idle sleep + display sleep)
    |
    +-- ClosedDisplayManager       Toggles `pmset disablesleep` via privileged helper script
    |       |
    |       +-- ShellExecutor      Protocol for running shell commands (injectable for testing)
    |
    +-- HelperInstaller            Installs/uninstalls sudoers entry + helper script for closed-lid mode
    |
    +-- BatteryMonitor             Reads IOKit power source info, detects low battery
    |
    +-- PowerSourceMonitor         CFRunLoop-based listener for AC/battery power source changes
```

## Detection Model

The app uses a reactive, session-aware approach:

1. **Claude Code Hooks**: Claude Code is configured to trigger `active.js` and `idle.js` scripts on specific events (`UserPromptSubmit`, `PreToolUse`, `Stop`, `Elicitation`, etc.). These scripts manage session-specific state files in `~/.claude/caffeine_sessions/`.

2. **Session Monitoring**: `ClaudeHookMonitor` polls the `caffeine_sessions` directory every 5 seconds. If any session files exist, the Mac is kept awake. This supports multiple concurrent terminal sessions and avoids race conditions by using individual files per `session_id`.

3. **Auto-Resume Wrapper**: An experimental Python wrapper tracks "hit your limit" messages and maintains its own session file while waiting for a reset, ensuring the Mac stays awake specifically during the wait timer.

## Key Design Decisions

- **No third-party dependencies.** The entire app is built on Foundation, AppKit, IOKit, and UserNotifications. This keeps the binary small and the attack surface minimal.
- **Sendable-first concurrency.** `ClaudeHookMonitor` is a `Sendable` actor. The poll runs on a detached `Task` with `.utility` priority, results are applied on `@MainActor`.
- **Signal handler cleanup.** A `nonisolated(unsafe)` reference to `ClosedDisplayManager` allows SIGTERM/SIGINT/SIGHUP handlers to call `forceDisable()` before exit, ensuring `pmset disablesleep` is always reset.
- **Immutable data flow.** Poll results are captured in `PollSnapshot` value types. The `AppDelegate` applies state changes based on the snapshot — it never mutates shared state from background threads.
- **Testability via injection.** `ClosedDisplayManager` accepts a `ShellExecutor` protocol and a `checkHelperInstalled` closure.

## Source Files

| File | Lines | Role |
|------|-------|------|
| `ClaudeCaffeine.swift` | ~1050 | App entry point, `AppDelegate`, menu bar UI, poll loop, state transitions |
| `ClaudeHookMonitor.swift` | ~50 | Scans `~/.claude/caffeine_sessions/` for active sessions based on hook files |
| `HookInstaller.swift` | ~120 | Installs/uninstalls Node.js hook helpers and updates `settings.json` |
| `SleepAssertionManager.swift` | ~70 | Creates/releases `IOPMAssertion` for system and display sleep prevention |
| `ClosedDisplayManager.swift` | ~120 | State machine for `pmset disablesleep` toggling via privileged helper |
| `HelperInstaller.swift` | ~155 | Installs/uninstalls the sudoers entry and shell script for closed-lid mode |
| `BatteryMonitor.swift` | ~50 | Reads battery level and charging state from IOKit power sources |
| `PowerSourceMonitor.swift` | ~50 | Listens for AC/battery power source changes via `IOPSNotificationCreateRunLoopSource` |
| `AutoResumeManager.swift` | ~250 | Manages the Python PTY wrapper for limit-aware auto-resuming |

## Test Files

| File | What it tests |
|------|---------------|
| `BatteryMonitorTests.swift` | Battery level reading, low-battery threshold logic |
| `ClosedDisplayManagerTests.swift` | Enable/disable state machine, helper-not-installed guard, force disable |
| `HelperInstallerTests.swift` | Script writing, sudoers validation, install/uninstall flows |

## How to Work on This Project

### Build & Run
```bash
swift build           # Debug build
swift build -c release  # Release build (.build/release/ClaudeCaffeine)
swift run             # Build + run
swift test            # Run all tests
```

### App Bundle
```bash
./scripts/make-app-bundle.sh    # Creates dist/Claude Caffeine.app
```

### Important Constraints

- **No `console.log` equivalent** — Use `os.Logger` or remove debug prints before committing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmslau/claude-caffeine](https://github.com/jmslau/claude-caffeine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
