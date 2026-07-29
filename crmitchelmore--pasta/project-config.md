---
trigger: always_on
description: Pasta is a macOS clipboard history manager built with SwiftUI and Swift Package Manager. It features a main app window and a Spotlight-style quick search panel.
---

# Pasta - Copilot Instructions

## Project Overview
Pasta is a macOS clipboard history manager built with SwiftUI and Swift Package Manager. It features a main app window and a Spotlight-style quick search panel.

## Architecture

### Key Components
- **PastaApp** (`Sources/PastaApp/`) - Main application, window management, hotkey handling
- **PastaCore** (`Sources/PastaCore/`) - Business logic, clipboard monitoring, database, search
- **PastaUI** (`Sources/PastaUI/`) - Reusable SwiftUI views and components
- **PastaDetectors** (`Sources/PastaDetectors/`) - Content type detection (URLs, emails, etc.)

### Window Types
- **MainWindow** - Standard resizable window for browsing clipboard history
- **QuickSearchWindow** - Floating panel (NSPanel) for quick Spotlight-style search

## Critical Patterns

### Search Implementation (FTS5)
**ALWAYS** use SQLite FTS5 for search, not in-memory fuzzy search libraries.

The database has an FTS5 virtual table (`clipboard_entries_fts`) with triggers to keep it synced.
Use `DatabaseManager.searchFTS()` which supports prefix matching:

```swift
// Fast: FTS5 search (<1ms for 10k+ entries)
let results = try database.searchFTS(query: "hello", contentType: nil, limit: 50)

// The query "hello world" becomes FTS5 query "hello* world*" for prefix matching
```

**Why:** In-memory Fuse search caused 200-500ms delays and beach ball on 6k+ entries.
FTS5 runs in SQLite's optimized C engine with inverted index.

**Also:** When filtering results on main thread, limit input to ~200 entries max:
```swift
let limited = Array(input.prefix(200))  // Prevent main thread blocking
```

### Quick Search Paste Behavior
**DO NOT** paste while the quick search panel is visible. The correct sequence is:

```swift
// 1. Hide the quick search panel first
quickSearchController?.hide()

// 2. Copy content to clipboard
pasteService.copy(entry)

// 3. Deactivate app to return focus to previous app
DispatchQueue.main.asyncAfter(deadline: .now() + 0.05) {
    NSApp.hide(nil)
    
    // 4. Small delay then simulate Cmd+V
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
        SystemPasteEventSimulator().simulateCommandV()
    }
}
```

**Why**: If you paste while the panel is visible, Cmd+V goes to our app instead of the user's previous app.

### Search Performance in SwiftUI
**NEVER** use computed properties for search results in SwiftUI views. This causes:
- Multiple recalculations per render (each access triggers search)
- Main thread blocking during typing
- Laggy/frozen UI

**DO** use this pattern:
```swift
// State for results
@State private var displayedEntries: [ClipboardEntry] = []
@State private var searchDebounceTask: Task<Void, Never>? = nil

// Debounced search trigger
.onChange(of: searchQuery) { _, newQuery in
    searchDebounceTask?.cancel()
    let trimmed = newQuery.trimmingCharacters(in: .whitespacesAndNewlines)
    
    if trimmed.isEmpty {
        displayedEntries = allEntries  // Immediate for empty
    } else {
        searchDebounceTask = Task {
            try? await Task.sleep(nanoseconds: 100_000_000)  // 100ms debounce
            guard !Task.isCancelled else { return }
            
            // Run search OFF main thread
            let results = await Task.detached(priority: .userInitiated) {
                // search logic here
            }.value
            
            await MainActor.run { displayedEntries = results }
        }
    }
}
```

### Sidebar / Filters Performance (SwiftUI)
Avoid O(N) work inside SwiftUI `View` computed properties (e.g. sidebar counts).
If counts/derived data depend on `entries`, precompute once when entries change
(`.onReceive(backgroundService.$entries)` or similar), store in `@State`, and pass into views.

For large datasets (5k+), also preload first-page results per filter (e.g. per ContentType)
off-main-thread so switching filters is instant.

### Initial History Load Performance
For large clipboard histories, preserve perceived launch speed:
- Publish a small first page immediately (currently 200 recent entries), then load the remaining history in bounded background pages.
- Show lightweight loading/progress feedback in the main window while history is still loading.
- Avoid duplicate startup refreshes between app launch and main-panel appearance.

### Source App Identity
When displaying or filtering source apps, canonicalise source identifiers before grouping:
- Treat bundle identifiers and display names for the same installed app as one identity.
- Keep every raw alias behind that identity so selecting a single sidebar row filters all matching entries.
- Use one shared resolver for app display names and icons across the sidebar, list rows, preview headers, and Quick Search, with fallbacks for imported names and Continuity.

### Copy Affordances Across Content Surfaces
When adding copy affordances for clipboard entries, cover both the list row and the preview/content pane. Users expect easy copying from the full content view as well as from each item in the list.

### Pathological Metadata Resilience
When entries contain large detector metadata (hundreds/thousands of matches), preserve interactivity first:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crmitchelmore/pasta](https://github.com/crmitchelmore/pasta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
