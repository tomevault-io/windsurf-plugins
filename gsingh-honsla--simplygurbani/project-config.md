---
trigger: always_on
description: - **Platform**: iOS 17+
---

# Project: Simply Gurbani

## Quick Reference
- **Platform**: iOS 17+
- **Language**: Swift 6.0
- **UI Framework**: SwiftUI
- **Architecture**: MVVM with @Observable
- **Minimum Deployment**: iOS 17.0
- **Package Manager**: Swift Package Manager + XcodeGen

## XcodeBuildMCP Integration
**IMPORTANT**: This project uses XcodeBuildMCP for all Xcode operations.
- Build: `mcp__xcodebuildmcp__build_sim_name_proj`
- Test: `mcp__xcodebuildmcp__test_sim_name_proj`
- Clean: `mcp__xcodebuildmcp__clean`
- Run xcodegen after project.yml changes: `xcodegen generate`

## Project Structure
```
SimplyGurbani/
├── App/                    # App entry point (SimplyGurbaniApp.swift, ContentView.swift)
├── Features/               # Feature modules
│   ├── Home/              # Hukamnama card, quick access grid
│   ├── Browse/            # Scripture, Bani, Raag lists
│   ├── Reader/            # Shabad/Bani reader views
│   ├── Search/            # Search and results
│   ├── Bookmarks/         # Saved verses, FolderBookmarksView
│   ├── Settings/          # Preferences
│   └── Feedback/          # User feedback submission
├── Core/
│   ├── Models/            # Domain models (Verse, Shabad, Bani, etc.)
│   ├── Networking/        # APIClient, Endpoints
│   ├── Persistence/       # SwiftData caching
│   └── Services/          # GurbaniService, BookmarkService
├── DesignSystem/
│   ├── Theme/             # AppTheme (Colors, Typography, Spacing)
│   ├── Components/        # GlassCard, GlassButton
│   └── Modifiers/         # GlassBackground, GurmukhiText
├── Navigation/            # AppRouter, Route, TabRoute
└── Resources/             # Assets, Fonts

docs/                       # GitHub Pages site
├── index.html             # Landing page
├── privacy.html           # Privacy policy
└── app-icon.png           # App icon for web

Screenshots/               # App Store screenshots
├── iPhone-6.7/           # iPhone 17 Pro Max (1320x2868)
└── iPad-11/              # iPad 11-inch (1668x2420)
```

## Coding Standards

### Swift Style
- Use Swift 6 strict concurrency
- Prefer `@Observable` over `ObservableObject`
- Use `async/await` for all async operations
- Follow Apple's Swift API Design Guidelines
- Use `guard` for early exits
- All models conform to `Sendable`

### SwiftUI Patterns
- Extract views when they exceed 80 lines
- Use `@State` for local view state only
- Use `@Environment` for dependency injection
- Use `NavigationStack` with type-safe `Route` enum
- Use `@Bindable` for bindings to @Observable objects
- Implement liquid glass design using `.glassBackground()` modifier

### Navigation Pattern
```swift
NavigationStack(path: $router.browsePath) {
    BrowseView()
        .navigationDestination(for: Route.self) { route in
            destinationView(for: route)
        }
}
```

### Bookmark System
- `BookmarkedVerse` model stores verse data, shabadID, optional baniID, and folder
- `BookmarkService` manages CRUD operations for bookmarks
- Bookmarks from banis include `baniID` for proper navigation
- Navigation routes include optional `scrollToVerseID` for scroll-to-verse
- Bookmark navigation prioritizes scroll position over saved reading position

### Reading Position Tracking
- `ReadingPosition` model tracks scroll position for shabads, banis, and angs
- `ReadingPositionService` manages reading history (max 20 items)
- iOS 17's `scrollPosition(id:)` API for automatic position tracking
- Continue Reading section shows last 3 reading sessions
- Position restored on navigation: bookmark position > saved position > top

## API Integration
- Base URL: `https://api.banidb.com/v2`
- Key endpoints:
  - `/shabads/:id` - Get shabad by ID
  - `/banis` - List all banis
  - `/banis/:id` - Get bani content
  - `/search/:query` - Search Gurbani
  - `/hukamnama/today` - Today's hukamnama
  - `/angs/:number` - Get ang/page
- All network calls go through `APIClient` actor
- Responses cached using SwiftData for offline access

### API Model Considerations
- **Be defensive with optionals**: BaniDB API may return `null` for fields that seem required
- `unicode` fields can be `null` - always use `String?` and provide fallback to `gurmukhi`
- Translation structures vary by language: `en` uses flat strings, `pu`/`es` use nested dicts
- `sourceId` may be missing from verse objects in some endpoints
- Test API responses with actual endpoint calls before finalizing Codable models

## Design System

### Color Palette
| Color | Hex | RGB | Usage |
|-------|-----|-----|-------|
| Light Beige | #FDF0D5 | (0.992, 0.941, 0.835) | App background |
| Dark Blue | #003049 | (0.0, 0.188, 0.286) | Primary text, headers, Quick Access tiles |
| Light Blue | #669BBC | (0.4, 0.608, 0.737) | Secondary accents, icons |
| Burgundy Red | #AB364D | (0.671, 0.212, 0.302) | Primary accent, buttons, highlights |
| Warm Ivory | #FFFEF7 | (1.0, 0.995, 0.97) | Card backgrounds |

### Components
- `GlassCard` - Card component with warm ivory background, subtle border, and shadow
- `GlassButtonStyle` - Button styles (`.glass` and `.glassProminent`)
- Use `GlassCard` for consistent card styling across all screens
- Gurmukhi fonts: GurbaniAkhar, AnmolLipi (register in Info.plist)

## Key Files
- `SimplyGurbani/App/SimplyGurbaniApp.swift` - App entry point with SwiftData container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gsingh-honsla/SimplyGurbani](https://github.com/gsingh-honsla/SimplyGurbani) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
