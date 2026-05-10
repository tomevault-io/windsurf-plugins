---
trigger: always_on
description: rule, roadmap, plan
---

# Implementation Roadmap for Anchor Project

## Development Phases

### Phase 1: Core Infrastructure ✅ (In Progress)
- [x] Package structure with AnchorKit shared module
- [x] Basic [Package.swift](mdc:Anchor/Package.swift) configuration
- [x] CLI entry point in [main.swift](mdc:Anchor/AnchorCLI/Sources/AnchorCLI/main.swift)
- [x] Menu bar app with [AnchorMenubarApp.swift](mdc:Anchor/Anchor/AnchorMenubarApp.swift)
- [x] Mobile app foundation with [AnchorMobileApp.swift](mdc:Anchor/AnchorMobile/AnchorMobileApp.swift)
- [ ] Swift ArgumentParser integration for CLI commands
- [ ] Basic command structure in [CLICommands.swift](mdc:Anchor/AnchorCLI/Sources/AnchorCLI/CLICommands.swift)

### Phase 2: Data Models & Storage
**Location**: `AnchorKit/Sources/AnchorKit/Models/`

#### Required Models:
- [x] [Place.swift](mdc:Anchor/AnchorKit/Sources/AnchorKit/Models/Place.swift) - Location data from Overpass API
- [x] [AuthCredentials.swift](mdc:Anchor/AnchorKit/Sources/AnchorKit/Models/AuthCredentials.swift) - SwiftData model for authentication
- [x] [AnchorSettings.swift](mdc:Anchor/AnchorKit/Sources/AnchorKit/Models/AnchorSettings.swift) - User preferences and defaults
- [x] [Item.swift](mdc:Anchor/AnchorMobile/Item.swift) - Mobile app SwiftData model example

#### Storage Implementation:
- [x] SwiftData ModelContainer setup for GUI apps
- [x] UserDefaults extensions for CLI credential storage
- [ ] Settings persistence and retrieval across platforms
- [ ] Migration handling for settings updates

### Phase 3: Service Layer
**Location**: `AnchorKit/Sources/AnchorKit/Services/`

#### Core Services:
- [x] [LocationService.swift](mdc:Anchor/AnchorKit/Sources/AnchorKit/Services/LocationService.swift) - CoreLocation wrapper with authorization
- [x] [OverpassService.swift](mdc:Anchor/AnchorKit/Sources/AnchorKit/Services/OverpassService.swift) - OpenStreetMap POI queries
- [x] [BlueskyService.swift](mdc:Anchor/AnchorKit/Sources/AnchorKit/Services/BlueskyService.swift) - AT Protocol authentication and posting
- [x] [CredentialsStorage.swift](mdc:Anchor/AnchorKit/Sources/AnchorKit/Services/CredentialsStorage.swift) - Platform-agnostic credential management

#### Service Features:
- [x] Async/await network operations
- [x] Proper error handling with custom error types
- [ ] Response caching for location queries
- [ ] Token refresh for Bluesky authentication

### Phase 4: Platform-Specific Implementation

#### CLI Commands Implementation
**Target**: AnchorCLI

##### `anchor login`
- [ ] Interactive Bluesky authentication
- [ ] Credential storage and validation
- [ ] Session management

##### `anchor settings`
- [ ] Default message configuration
- [ ] Settings display and modification
- [ ] Validation of user inputs

##### `anchor nearby`
- [ ] Current location detection
- [ ] Overpass API integration for POI search
- [ ] Optional text filtering (`--filter climbing`)
- [ ] Formatted output with place IDs

##### `anchor drop`
- [ ] Interactive mode (no arguments)
  - Location detection
  - Nearby POI selection
  - Optional message prompt
- [ ] Parameterized mode
  - `--place <type:id>` option
  - `--message <text>` option
- [ ] Bluesky post creation and submission

#### Menu Bar App Features
**Target**: Anchor (macOS)
**Location**: `Anchor/Features/`

##### Core Views:
- [x] [CheckInView.swift](mdc:Anchor/Anchor/Features/CheckIn/Views/CheckInView.swift) - Check-in functionality
- [x] [FeedTabView.swift](mdc:Anchor/Anchor/Features/Feed/Views/FeedTabView.swift) - Social feed display
- [x] [NearbyTabView.swift](mdc:Anchor/Anchor/Features/Nearby/Views/NearbyTabView.swift) - Nearby places discovery
- [x] [SettingsWindow.swift](mdc:Anchor/Anchor/Features/Settings/Views/SettingsWindow.swift) - App settings and preferences

##### Menu Bar Integration:
- [ ] Status bar item configuration
- [ ] Popover-based interface
- [ ] Keyboard shortcuts and accessibility

#### Mobile App Features
**Target**: AnchorMobile (iOS 26.0+)
**Location**: `AnchorMobile/`

##### Core Features:
- [x] [AnchorMobileApp.swift](mdc:Anchor/AnchorMobile/AnchorMobileApp.swift) - App entry point with SwiftData
- [x] [ContentView.swift](mdc:Anchor/AnchorMobile/ContentView.swift) - Main navigation interface
- [ ] Tab-based navigation (Check-in, Feed, Nearby, Settings)
- [ ] Responsive design for different iPhone screen sizes
- [ ] iOS-specific location permissions handling

##### Mobile-Specific Features:
- [ ] Pull-to-refresh for feeds
- [ ] Haptic feedback for interactions
- [ ] Background app refresh for location updates
- [ ] iOS sharing integration for check-ins

### Phase 5: Testing & Validation
**Testing Framework**: Swift Testing (not XCTest)

#### Test Coverage:
**AnchorKit Tests** (`AnchorKit/Tests/`):
- [x] Unit tests for all Models using Swift Testing
- [x] Service layer tests with mocked network calls
- [x] Protocol-based mocking to avoid SwiftData ModelContainer issues
- [ ] Error handling validation
- [ ] Location service permission testing

**Platform-Specific Tests**:
- [ ] CLI command integration tests (`AnchorTests/`)
- [ ] Menu bar app UI tests (`AnchorTests/`)
- [x] Mobile app unit tests (`AnchorMobileTests/`)
- [x] Mobile app UI tests (`AnchorMobileUITests/`)

### Phase 6: Polish & Documentation
- [ ] Comprehensive error messages across all platforms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dropanchorapp/Anchor](https://github.com/dropanchorapp/Anchor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
