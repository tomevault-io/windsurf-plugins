---
trigger: always_on
description: A skim-first orientation for future sessions. Code is the source of truth — this file captures the things that aren't obvious from reading source: conventions, gotchas, and the release flow.
---

# MFinder — Working Notes

A skim-first orientation for future sessions. Code is the source of truth — this file captures the things that aren't obvious from reading source: conventions, gotchas, and the release flow.

## Project shape

- Native macOS Windows-Explorer-style file manager. SwiftUI shell with AppKit views in the spots where SwiftUI's NSTextField/focus model is unreliable (sidebar tree, details file table).
- Korean UI throughout. Sidebar sections: **즐겨찾기 / 내 PC / 네트워크**.
- Repo: `github.com/secondlook-hub/MFinder` (public). `gh` CLI is authenticated for that org.
- Min macOS 13. Swift 5.9. Single executable target via Package.swift.

## Build & release flow

```bash
./build.sh release             # → MFinder.app at repo root
scripts/makeDmg.sh             # → MFinder-v{X}.dmg + MFinder.dmg (Applications symlink)
```

### Releasing v{X.Y}

1. `Resources/Info.plist` — bump `CFBundleVersion` + `CFBundleShortVersionString` (always change both).
2. `README.md` — extend Features list if user-visible.
3. Commit: `git commit -m "<subject> (v{X.Y})"` with HEREDOC body. Push: `git push`.
4. `osascript -e 'tell application "MFinder" to quit'` (running MFinder holds the old binary), `./build.sh release`, `scripts/makeDmg.sh`.
5. `gh release create v{X.Y} MFinder-v{X.Y}.dmg MFinder.dmg --title "MFinder v{X.Y}" --notes "..."`.
6. `open https://github.com/secondlook-hub/MFinder/releases/tag/v{X.Y}` to show the user.

The in-app updater (Help → "업데이트 확인…") fetches the latest release via the GitHub API. It prefers an asset literally named `MFinder.dmg` (the stable alias), otherwise the first `.dmg`. Existing users get a Download alert on next launch.

### Commit / release-notes style

- Subject ends with ` (v{X.Y})` in the commit, not in PR titles. HEREDOC body explains *why*.
- Release notes use plain markdown headers like `## What's New`. Korean for menu/feature labels, English for narrative.

## Architecture cheat sheet

```
Sources/MFinder/
├── MFinderApp.swift        App entry, menu commands, AppDelegate.
│                           Global NSEvent.addLocalMonitorForEvents for
│                           F2/Space/⌫/⌘⌫/⌘⇧⌫/⌥⌘⌫ and (sidebar-only)
│                           ⌘C/⌘X/⌘V. applicationDockMenu adds "새 창 열기".
│                           launchNewMFinderInstance() spawns a new process
│                           via NSWorkspace.openApplication(... configuration
│                           with createsNewApplicationInstance = true).
├── ContentView.swift       Tabs root. ⚠ SidebarView is tagged
│                           `.id(ObjectIdentifier(tab))` so it rebuilds when
│                           the active tab changes (the Coordinator's Combine
│                           subscriptions are scoped to one nav/tree instance).
│                           Hosts the update-checker alert.
├── Models/
│   ├── NavigationState.swift   Per-tab. Notable fields:
│   │     - pendingSelection: Set<URL>? — sticky across reload generations
│   │       so an FSEvent-triggered reload arriving right after a paste
│   │       still applies the original `thenSelect:` set. Cleared on
│   │       navigate/back/forward.
│   │     - sidebarSelectionURLs: [URL] — published mirror of NSOutlineView
│   │       multi-selection. Sidebar Cmd+C/X/⌘⌫ handlers read this; falls
│   │       back to currentURL when empty.
│   │     - reload(thenSelect:) uses loadGeneration to cancel obsolete
│   │       async reloads; the FSEvent watcher skips while renamingURL set.
│   ├── TabsState.swift              Tab collection.
│   ├── FolderTreeStore.swift        Per-tab. expandedURLs + childrenCache.
│   │                                ensureVisible(_:) walks ancestors;
│   │                                loadChildren is sync (cheap FS read).
│   └── FileItem.swift
├── Services/
│   ├── ClipboardService.swift    Singleton, in-process. Stack semantics
│   │     (cut/copy accumulate; ⌘V drains). hasContent also queries
│   │     NSPasteboard so a fresh second instance can paste. didBecomeActive
│   │     bumps pasteboardSnapshot (@Published) so SwiftUI re-evaluates.
│   ├── ArchiveService.swift      Detects unrar/unar/7z/tar/etc. at standard
│   │     Homebrew paths. bandizipAppURL resolved via bundle id
│   │     com.bandisoft.mac.bandizip. RAR/7z fall back to
│   │     openWithBandizip(_:) when no CLI is installed.
│   ├── FileSystemWatcher.swift   kqueue vnode source per watched dir.
│   ├── FileSystemService.swift   list/quickAccessLocations/thisPCLocations/
│   │                              moveToTrash (AppleScript Finder).
│   ├── PinnedFoldersService.swift  UserDefaults; @Published pinnedURLs.
│   ├── UpdateChecker.swift       GitHub Releases poll, semver compare.
│   └── (PreferencesService, SearchSnippetService, QuickLookCoordinator)
└── Views/
    ├── SidebarView.swift             Thin SwiftUI host: SidebarOutlineRepresentable
    │                                 + .onReceive notification handlers for
    │                                 sidebar-scoped Cmd+C/X/V/⌘⌫/F2.
    ├── SidebarOutlineView.swift      NSOutlineView wrapper for the ENTIRE

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [secondlook-hub/MFinder](https://github.com/secondlook-hub/MFinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
