---
trigger: always_on
description: > Read this **before touching code**. It captures architecture decisions, hard rules,
---

# AGENTS.md — Blur-Browser

> Read this **before touching code**. It captures architecture decisions, hard rules,
> and the known-gotcha list that keeps this app from breaking when you make changes.

---

## 1. What this is

**Blur-Browser** — a native macOS browser. SwiftUI for leaf views, AppKit for
everything structural (window chrome, sidebar, toolbar, `WKWebView` hosting,
overlays). Built for macOS 14+, Swift 5.10+.

- Xcode project: `Blur-Browser.xcodeproj`
- Scheme: `Blur-Browser`
- Bundle ID: `com.Blur-Browser.app`
- Single SPM dependency: a private `AsyncImage` library (github.com/EngOmarElsayed/AsyncImage) — though the source is **also vendored** in `Browse/AsyncImage/` for favicon rendering. Do not confuse the two.

### Build & run

```bash
xcodebuild -project Blur-Browser.xcodeproj -scheme Blur-Browser -configuration Debug build
```

The project uses Xcode's "Synchronized Folder" system — Swift files added to
`Browse/` are auto-included in the target (see `fileSystemSynchronizedGroups`
in `project.pbxproj`). **You do NOT need to add new files to the Xcode project
manually.** Just `Write` the file under `Browse/…`.

Exceptions are listed in `PBXFileSystemSynchronizedBuildFileExceptionSet`
(currently: `Assets.xcassets`, `Resources/Info.plist`).

---

## 2. Hard architectural rules

Violating any of these will silently break things or crash.

### AppKit hosts SwiftUI — never the other way around

- The window's content view controller is an AppKit `NSViewController`
  (`MainSplitViewController`)
- SwiftUI views are embedded as subviews via `NSHostingController`
- Manual frame-based layout for the split view — **no `NSSplitViewController`**
  (it causes infinite layout passes with `NSHostingController`)
- **No `NSToolbar` with custom view items** — same layout-cycle issue
- **Never set `NSHostingView` directly as `self.view`** on a controller —
  always wrap in a plain `NSView` container

### State management

- `@Observable` + `@MainActor` for every state class. **No `ObservableObject` / `@StateObject`.**
- `TabManager` is the single source of truth for tab state. All mutations go
  through its methods. Do not manipulate `tabs` array from anywhere else.
- Web state lives on `BrowserTab` — one `WKWebView` per tab, stored at
  `tab.webView`. Only the selected tab's web view is in the view hierarchy.

### Keyboard shortcuts

- All shortcuts are declared in `Browse/App/AppMenuBuilder.swift`
- **Every menu item targeting `AppDelegate` must have `target = delegate` set
  explicitly**. Otherwise shortcuts fail when `WKWebView` is first responder
  (the responder chain doesn't reach `AppDelegate` because web views consume
  events).
- For shortcuts that must work even when an in-process web window has focus
  (e.g., ⌘⌥C for Web Inspector), override `NSApplication.sendEvent(_:)` —
  see `Browse/App/main.swift` (`BrowserApplication`).

### Layout cycle prevention

The combo `fullSizeContentView` + `NSToolbar` + `NSSplitViewController` +
`NSHostingController` causes infinite layout passes on macOS. This project
avoids it by:
- Using a plain `NSViewController` (`MainSplitViewController`) with manual
  `frame` layout in `layoutSubviews()`
- No `NSToolbar` — the address bar is a regular view embedded in the
  content area
- `BrowserWindow` uses `.titled + .closable + .miniaturizable + .resizable
  + .fullSizeContentView` but **no** `.unifiedTitleAndToolbar`

### No 3rd party dependencies

Beyond the one vendored `AsyncImage`, do not add SPM/Cocoapods/Carthage
packages. Everything else is Foundation/AppKit/SwiftUI/WebKit/SwiftData.

---

## 3. Folder map

```
Browse/
├── App/
│   ├── main.swift                       # NSApplication subclass that intercepts ⌘⌥C
│   ├── AppDelegate.swift                # App lifecycle + forwards all actions to BrowserWindowController
│   └── AppMenuBuilder.swift             # Full menu bar with ALL keyboard shortcuts
│
├── Window/
│   ├── BrowserWindow.swift              # NSWindow subclass — also swallows unhandled keyDown to prevent beep
│   ├── BrowserWindowController.swift    # Owns TabManager, HistoryStore, DownloadStore, DownloadManager
│   └── MainSplitViewController.swift    # ROOT view controller — sidebar + toolbar + web view + overlays
│
├── Sidebar/
│   ├── SidebarViewController.swift      # AppKit host for the SwiftUI SidebarView
│   └── TabsSideBarView/
│       ├── SidebarView.swift            # SwiftUI root — holds SidebarState, Tabs vs Downloads
│       └── SubViews/
│           ├── SidebarContentView.swift  # Switches between Tabs list & Downloads list
│           ├── SidebarButtons.swift      # Bottom action row (Home / Downloads / History / Settings)
│           ├── PinnedTabsGrid.swift      # Pinned tabs section (top)
│           ├── PinnedTabItemView.swift   # Single pinned tab
│           ├── UnpinnedTabsList.swift    # Regular tab list
│           └── TabItemView.swift         # Single unpinned tab
│
├── Tab/
│   ├── BrowserTab.swift                 # @Observable — id, url, title, webView, isPinned, readerArticle, browsingError, etc.
│   ├── TabManager.swift                 # @Observable — source of truth. add/close/pin/unpin/move/navigate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EngOmarElsayed/blur-browser](https://github.com/EngOmarElsayed/blur-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
