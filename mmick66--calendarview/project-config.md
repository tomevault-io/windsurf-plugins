---
trigger: always_on
description: This file provides instructions and context for AI coding agents working on this project.
---

# Project Instructions for AI Agents

This file provides instructions and context for AI coding agents working on this project.

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:1105d646 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/core-concepts/sync-concepts.md for details and anti-patterns.

## Agent Context Profiles

The managed Beads block is task-tracking guidance, not permission to override repository, user, or orchestrator instructions.

- **Conservative (default)**: Use `bd` for task tracking. Do not run git commits, git pushes, or Dolt remote sync unless explicitly asked. At handoff, report changed files, validation, and suggested next commands.
- **Minimal**: Keep tool instruction files as pointers to `bd prime`; use the same conservative git policy unless active instructions say otherwise.
- **Team-maintainer**: Only when the repository explicitly opts in, agents may close beads, run quality gates, commit, and push as part of session close. A current "do not commit" or "do not push" instruction still wins.

## Session Completion

This protocol applies when ending a Beads implementation workflow. It is subordinate to explicit user, repository, and orchestrator instructions.

1. **File issues for remaining work** - Create beads for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **Handle git/sync by active profile**:
   ```bash
   # Conservative/minimal/default: report status and proposed commands; wait for approval.
   git status

   # Team-maintainer opt-in only, unless current instructions forbid it:
   git pull --rebase
   git push
   git status
   ```
5. **Hand off** - Summarize changes, validation, issue status, and any blocked sync/commit/push step

**Critical rules:**
- Explicit user or orchestrator instructions override this Beads block.
- Do not commit or push without clear authority from the active profile or the current user request.
- If a required sync or push is blocked, stop and report the exact command and error.
<!-- END BEADS INTEGRATION -->


## Build & Test

Requires Xcode 26 and an iOS 26 simulator. CI (`.github/workflows/ci.yml`) runs the same four steps.

```bash
# Test the package (Swift Testing, KDCalendarTests)
xcodebuild -scheme KDCalendar-Package -destination 'platform=iOS Simulator,name=iPhone 17 Pro' test

# Build the DocC documentation for both products
xcodebuild docbuild -scheme KDCalendar -destination 'platform=iOS Simulator,name=iPhone 17 Pro'
xcodebuild docbuild -scheme KDCalendarEventKit -destination 'platform=iOS Simulator,name=iPhone 17 Pro'

# Build the example app
xcodebuild -project Example/KDCalendarDemo.xcodeproj -scheme KDCalendarDemo \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro' CODE_SIGNING_ALLOWED=NO build

# Lint (must pass with no warnings)
xcrun swift-format lint --strict --recursive Sources Tests Example/KDCalendarDemo
scripts/lint-open-documentation.sh   # swift-format skips `open` declarations
```

When several worktrees test at once, give each its own simulator (see `bd memories simulator`):
`xcrun simctl create <name> 'iPhone 17 Pro'`, test with `-destination id=<udid>`, delete it afterwards.

The demo app (`com.karmadust.KDCalendarDemo`) takes two launch arguments: `-tab N` opens tab N
(0 classic, 1 default style, 2 SwiftUI) and `-sampleEvents` seeds events without asking for
calendar access. For example `xcrun simctl launch booted com.karmadust.KDCalendarDemo -tab 2 -sampleEvents`.

## Architecture Overview

A Swift package (`Package.swift`, Swift 6 language mode, iOS 17+) with two products; CocoaPods
mirrors them as the `Core` and `EventKit` subspecs of `KDCalendar.podspec`.

- **`KDCalendar`** (`Sources/KDCalendar`): the calendar view, UIKit first.
  - `CalendarView.swift`: `CalendarView` (a `UIView` around a paging `UICollectionView`): its
    state, setup, layout and right-to-left handling, and the grid it reads from (`reloadData()`,
    `refreshSnapshot()`).
  - `CalendarView+Selection.swift`: the public selection API and the collection view's selection
    callbacks. `CalendarView+Scrolling.swift`: `setDisplayDate`, the next and previous month, the
    scroll callbacks and the month notification. `CalendarView+DataSource.swift`: the cells.
  - `CalendarEvent.swift`, `CalendarViewDataSource.swift` and `CalendarViewDelegate.swift`: the
    event type and the two protocols with their default implementations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmick66/CalendarView](https://github.com/mmick66/CalendarView) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
