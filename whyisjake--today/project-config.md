---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working with code in this repository.
---

# GitHub Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

## Project Overview

"Today" is a SwiftUI iOS application that serves as an RSS reader with AI-powered content summarization. It uses SwiftData for persistent storage, Apple's NaturalLanguage framework and Apple Intelligence for text analysis, and supports background fetch for automatic feed syncing.

**Key Technologies:**
- SwiftUI for declarative UI
- SwiftData for persistence (successor to Core Data)
- Swift Concurrency (async/await)
- Apple Intelligence (iOS 26+) with NaturalLanguage framework fallback
- Background Tasks (BGTaskScheduler)
- Safari View Controller for in-app browsing

## Build and Development Commands

### Building the Project

```bash
# Build for debug
xcodebuild -project Today.xcodeproj -scheme Today -configuration Debug build

# Build for release
xcodebuild -project Today.xcodeproj -scheme Today -configuration Release build

# Clean build artifacts
xcodebuild -project Today.xcodeproj -scheme Today clean

# Open in Xcode
open Today.xcodeproj
```

### Running Tests

**Important**: Tests require an iOS Simulator. The project must be run on macOS with Xcode installed.

```bash
# Run all tests
xcodebuild test -project Today.xcodeproj -scheme Today -destination 'platform=iOS Simulator,name=iPhone 15'

# Run specific test suite
xcodebuild test -project Today.xcodeproj -scheme Today -destination 'platform=iOS Simulator,name=iPhone 15' -only-testing:TodayTests/TexturizerTests

# List available simulators
xcrun simctl list devices available
```

**Available Test Suites:**
- `TodayTests/RSSParserTests` - RSS feed parsing tests
- `TodayTests/AtomFeedTests` - Atom feed format tests
- `TodayTests/JSONFeedTests` - JSON feed format tests
- `TodayTests/RedditRSSTests` - Reddit JSON API tests
- `TodayTests/TexturizerTests` - Typography/text processing tests
- `TodayTests/CategoryManagerTests` - Category management tests
- `TodayTests/ConditionalHTTPClientTests` - HTTP client tests
- `TodayTests/HTMLHelperTests` - HTML parsing helper tests

### Version Management

```bash
# Update marketing version
xcrun agvtool new-marketing-version 1.3

# Increment build number
xcrun agvtool next-version -all
```

## Project Structure

```
Today/
├── Models/           # SwiftData models (Feed, Article)
├── Views/            # SwiftUI views
├── Services/         # Business logic (RSS parsing, AI, background sync)
├── Utilities/        # Helper utilities (Texturizer)
└── Resources/        # Assets and configuration
```

## Architecture

### Data Layer - SwiftData

Models are located in `Today/Models/`:

- **Feed.swift**: RSS feed subscriptions with title, URL, category, and relationship to articles. Uses `@Relationship(deleteRule: .cascade)` to auto-delete articles when feed is deleted.
- **Article.swift**: Individual RSS articles with metadata (title, link, description, published date, author, guid). Includes `isRead`, `isFavorite`, and `aiSummary` properties.
- **ModelContainer**: Initialized in `TodayApp.swift` with schema containing `Feed` and `Article`. Configured for persistent storage.

### Service Layer

Services are located in `Today/Services/`:

- **RSSParser.swift**: XMLParser-based RSS feed parser. Handles RSS and Atom formats with multiple date format support. Returns parsed article data without direct database access.
- **FeedManager.swift**: `@MainActor` class managing feed subscriptions and syncing. Handles duplicate detection using article GUIDs. Owns ModelContext for database operations.
- **AIService.swift**: Uses Apple's NaturalLanguage framework for content analysis. Provides article summarization, keyword extraction, trend analysis, and conversational responses.
- **OnDeviceAIService.swift**: Uses Apple Intelligence (iOS 26+) for advanced AI summaries with graceful fallback to NaturalLanguage framework.
- **BackgroundSyncManager.swift**: Manages `BGAppRefreshTask` for background feed syncing. Registers background tasks on app launch and schedules periodic syncs (minimum 15 min intervals).

### View Layer

Views are located in `Today/Views/`:

- **TodayView.swift**: Main article list with day-based navigation (Today/Yesterday/Older), category filtering, and search. Uses `@Query` with sort descriptors for reactive data updates.
- **FeedListView.swift**: Feed management interface with add/remove/sync capabilities. Uses `@StateObject` for FeedManager lifecycle.
- **AIChatView.swift**: Chat-style interface for AI interactions. Maintains conversation history with `ChatMessage` structs.
- **ArticleDetailSimple.swift**: Article detail view with toolbar actions (share, favorite, mark read).
- **SettingsView.swift**: User preferences including theme, accent color, font selection, and AI provider.

### App Structure

- **TodayApp.swift**: Entry point that initializes BackgroundSyncManager and ModelContainer. Schedules background fetch on app launch.
- **ContentView.swift**: TabView-based navigation between Today, Feeds, and AI Summary tabs.

## Key Development Patterns

### SwiftData Usage

- **Access model context**: Use `@Environment(\.modelContext)` in views
- **Query data**: Use `@Query` with predicates and sort descriptors
  ```swift

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whyisjake/today](https://github.com/whyisjake/today) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
