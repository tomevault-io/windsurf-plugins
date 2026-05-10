---
trigger: always_on
description: This project uses Swift 6 with strict concurrency enabled across all targets. Follow these patterns for modern, safe concurrent code. See [swift-concurrency.md](mdc:Anchor/Anchor/Docs/swift-concurrency.md) for complete Swift 6 concurrency guide.
---

# Swift Development Patterns for Anchor Project

## Swift 6 Conventions

This project uses Swift 6 with strict concurrency enabled across all targets. Follow these patterns for modern, safe concurrent code. See [swift-concurrency.md](mdc:Anchor/Anchor/Docs/swift-concurrency.md) for complete Swift 6 concurrency guide.

### Concurrency
- Use `async/await` for all network operations
- Use `Sendable` protocol for data models that cross concurrency boundaries
- Follow strict concurrency checking for data race prevention
- Use actors for protecting mutable state
- Apply `@MainActor` for UI-related code

### Project Structure
- **CLI Target**: [AnchorCLI/Sources/AnchorCLI/main.swift](mdc:Anchor/Anchor/AnchorCLI/Sources/AnchorCLI/main.swift)
- **Menu Bar App**: [Anchor/AnchorMenubarApp.swift](mdc:Anchor/Anchor/Anchor/AnchorMenubarApp.swift)
- **Mobile App**: [AnchorMobile/AnchorMobileApp.swift](mdc:Anchor/Anchor/AnchorMobile/AnchorMobileApp.swift)
- **Shared Library**: AnchorKit module for reusable components
- **Dependencies**: Defined in [Package.swift](mdc:Anchor/Anchor/Package.swift) with local AnchorKit package

## MV Pattern Architecture

Follow the **MV Pattern** (Model-View without ViewModels) as outlined in [mv_pattern_guidelines.md](mdc:Anchor/Anchor/Docs/mv_pattern_guidelines.md):

### Core Principles
- **No ViewModels** - Use `@Observable` classes for business logic
- **Views own UI state** - Use `@State` for temporary UI concerns  
- **Stores handle operations** - Create focused `@Observable` classes
- **Environment for sharing** - Use `@Environment` across view hierarchies

### Property Wrapper Guide

| Use Case | Wrapper | Example |
|----------|---------|---------|
| Create store | `@State` | `@State private var store = AnchorStore()` |
| Share store | `@Environment` | `@Environment(AnchorStore.self) private var store` |
| UI state | `@State` | `@State private var showingSheet = false` |
| Settings | `@AppStorage` | `@AppStorage("theme") var theme = "light"` |
| SwiftData display | `@Query` | `@Query private var checkins: [CheckIn]` |

### Code Organization

#### Feature-Based Organization
```
AppName/Features/
├── CheckIn/
│   └── Views/          ← SwiftUI views with @State for UI
├── Feed/
│   └── Views/          ← Display views using @Query + @Environment
├── Nearby/
│   └── Views/          ← Nearby places views
└── Settings/
    └── Views/          ← Settings interface
```

#### Models
**Shared Models** in `AnchorKit/Sources/AnchorKit/Models/`:
- [Place.swift](mdc:Anchor/Anchor/AnchorKit/Sources/AnchorKit/Models/Place.swift) - Location data from Overpass API
- [AuthCredentials.swift](mdc:Anchor/Anchor/AnchorKit/Sources/AnchorKit/Models/AuthCredentials.swift) - Bluesky authentication (SwiftData @Model)
- [AnchorSettings.swift](mdc:Anchor/Anchor/AnchorKit/Sources/AnchorKit/Models/AnchorSettings.swift) - User preferences

#### Services  
Place service classes in `AnchorKit/Sources/AnchorKit/Services/`:
- `BlueskyService.swift` - AT Protocol communication
- `OverpassService.swift` - OpenStreetMap queries
- `LocationService.swift` - CoreLocation wrapper
- `CredentialsStorage.swift` - Platform-agnostic credential management

#### Stores (Minimal Business Logic)
For Anchor's simple use case, place one main store in `AnchorKit/Sources/AnchorKit/Stores/`:

```swift
@Observable
class AnchorStore {
    // State
    private(set) var isCreatingCheckIn = false
    private(set) var isLoadingNearby = false
    private(set) var currentLocation: CLLocation?
    private(set) var nearbyPlaces: [Place] = []
    private(set) var error: Error?
    
    // Services
    private let locationService: LocationService
    private let blueskyService: BlueskyService
    private let overpassService: OverpassService
    
    init(locationService: LocationService, blueskyService: BlueskyService, overpassService: OverpassService) {
        self.locationService = locationService
        self.blueskyService = blueskyService
        self.overpassService = overpassService
    }
    
    func createCheckIn(at place: Place? = nil, message: String? = nil) async throws {
        isCreatingCheckIn = true
        defer { isCreatingCheckIn = false }
        
        // Get location if not provided
        let location = place?.coordinate ?? (try await locationService.getCurrentLocation())
        
        // Post to Bluesky
        try await blueskyService.postCheckIn(location: location, message: message)
    }
    
    func loadNearbyPlaces() async throws {
        isLoadingNearby = true
        defer { isLoadingNearby = false }
        
        let location = try await locationService.getCurrentLocation()
        nearbyPlaces = try await overpassService.searchNearby(location: location)
    }
}
```

## SwiftUI Patterns

Follow modern SwiftUI patterns as documented in [SwiftUI-api-20250627.md](mdc:Anchor/Anchor/Docs/SwiftUI-api-20250627.md):

### Store Creation and Sharing

```swift
// App Setup - Minimal store creation
@main
struct AnchorApp: App {
    @State private var anchorStore = AnchorStore()
    
    var body: some Scene {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dropanchorapp/Anchor](https://github.com/dropanchorapp/Anchor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
