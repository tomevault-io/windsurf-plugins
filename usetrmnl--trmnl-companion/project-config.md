---
trigger: always_on
description: This is a Calendar Sync iOS app built for TRMNL that fetches calendar events via EventKit and syncs them to a remote service. The app is designed to run on iOS 17.0+ and uses SwiftUI with SwiftData for persistence.
---

# CLAUDE.md - AI Assistant Project Context

## Project Overview
This is a Calendar Sync iOS app built for TRMNL that fetches calendar events via EventKit and syncs them to a remote service. The app is designed to run on iOS 17.0+ and uses SwiftUI with SwiftData for persistence.

## Key Implementation Details

### Architecture
- **MVVM Pattern**: Views → ViewModels → Services → Models
- **Singleton Services**: CalendarService and APIService use shared instances
- **SwiftData**: Used for sync history persistence
- **@Observable macro (iOS 17+)**: Modern observation framework replacing ObservableObject
- **Result Types**: Type-safe error handling with Result<Success, Error> pattern
- **ErrorKit Integration**: Consistent user-friendly error messages via Throwable protocol

### Error Handling System
The app uses typed errors with ErrorKit for comprehensive error handling.

Read https://github.com/FlineDev/ErrorKit to know more about ErrorKit usage

```swift
// API Layer - Strongly typed network/HTTP errors
enum APIError: Throwable {
    case invalidURL
    case encodingFailed(String)
    case networkError(NetworkErrorType)
    case httpError(HTTPErrorType)
    case invalidResponse
    
    var userFriendlyMessage: String { ... }
}

// Service Layer - Domain-specific errors that wrap lower-level errors
enum SyncError: Throwable {
    case noCalendarsSelected
    case noEventsFound
    case apiError(APIError)  // Composition pattern
    case persistenceError(String)
    
    var userFriendlyMessage: String { ... }
}
```

#### Result Pattern Usage
```swift
// API Service returns Result with event count on success
func syncEvents(_ events: [EventModel]) async -> Result<Int, APIError>

// Sync Service returns Result with rich success data
func performSync(...) async -> Result<SyncSuccess, SyncError>

// Success type carries meaningful data
struct SyncSuccess {
    let eventCount: Int
    let syncDuration: TimeInterval
    let timestamp: Date
}
```

#### Error Propagation Flow
1. Network/HTTP errors originate in APIService as `APIError`
2. SyncService wraps API errors and adds domain-specific errors as `SyncError`
3. ViewModels extract user-friendly messages via `error.userFriendlyMessage`
4. UI displays consistent, helpful error messages to users

### Calendar Identifier Strategy
```swift
// In EventModel.swift
self.calendarIdentifier = event.calendarItemExternalIdentifier ?? event.calendarItemIdentifier
```
- Primary: `calendarItemExternalIdentifier` - Best for cross-device consistency
- Fallback: `calendarItemIdentifier` - Stable on device
- **Known Issue**: Apple warns about possible duplicates with external IDs
- **Solution**: Remote service should deduplicate using combination of calendar_identifier + start_full + summary

### Memory Management
- Single `EKEventStore` instance maintained in CalendarService
- Event enumeration instead of array building for memory efficiency
- Date range hardcoded: today -6 to today +30 days

## @Observable Migration (iOS 17+)

### Why We Migrated
The app was migrated from `ObservableObject` to the new `@Observable` macro to simplify observation and avoid data flow issues, such as with nested objects.

### What Changed
```swift
// Before: ObservableObject + @Published
class SomeViewModel: ObservableObject {
    @Published var property = value
}

// After: @Observable
@Observable
class SomeViewModel {
    var property = value  // No @Published needed
}
```

### View Updates
- `@StateObject` → `@State` for owned objects
- `@ObservedObject` → Direct reference or `@Bindable` for two-way binding
- Automatic observation of nested objects

### Key Benefits
- Cleaner code with less boilerplate
- Automatic nested object observation
- Better performance (only observes used properties)
- No manual Combine subscriptions needed

## API Integration

### Production Endpoints
- **Base URL**: `https://usetrmnl.com/api`
- **Authentication**: Bearer token with API key in Authorization header

### Core Endpoints

#### 1. GET /me
- Validates API key and retrieves user information
- Returns: User object with name, email, timezone, API key
- Used for: Initial authentication validation

#### 2. GET /plugin_settings?plugin_id=58
- Fetches plugin settings for calendar plugin (ID: 58)
- Returns: Array of PluginSetting objects with id, name, pluginId
- Used for: Getting plugin setting IDs for data updates

#### 3. POST /plugin_settings/{id}/data
- Updates plugin with calendar events
- Payload structure:
```json
{
  "merge_variables": {
    "events": [
      // Array of EventModel objects
    ]
  }
}
```
- Returns: Success status
- Used for: Syncing calendar events to TRMNL

### Plugin System
- **Plugin ID 58**: Hardcoded calendar plugin identifier
- **One Request Per Plugin**: Events from multiple calendars are merged before sending
- **Plugin Mapping**: Calendars are mapped to plugins via PluginMappingManager
- **Persistence**: Plugin mappings stored in UserDefaults

## Authentication System

### API Key Storage
- **Storage**: AppStorage (UserDefaults wrapper) with key `"TRMNLAPIKey"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usetrmnl/trmnl-companion](https://github.com/usetrmnl/trmnl-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
