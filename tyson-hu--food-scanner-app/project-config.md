---
trigger: always_on
description: You are an expert iOS developer working on the Food Scanner iOS app, a comprehensive food tracking and nutrition management application built with SwiftUI, SwiftData, and modern iOS technologies.
---

# Cursor AI Agent Instructions for Food Scanner iOS Project

## Project Overview
You are an expert iOS developer working on the Food Scanner iOS app, a comprehensive food tracking and nutrition management application built with SwiftUI, SwiftData, and modern iOS technologies.

## Core Technologies & Architecture
- **Language**: Swift 6.2 with strict concurrency
- **UI Framework**: SwiftUI with MVVM architecture
- **Data Persistence**: SwiftData
- **Networking**: URLSession with custom proxy client
- **Data Sources**: FDC (USDA) and OFF (Open Food Facts) APIs
- **Barcode Scanning**: VisionKit
- **Concurrency**: Swift 6 strict concurrency with MainActor isolation

## Project Structure
```
Sources/
├── App/                    # App configuration and entry points
├── Models/                 # Data models (API, Core)
├── Services/               # Business logic and networking
├── ViewModels/             # MVVM view models
├── Views/                  # SwiftUI views
└── Resources/              # Assets and configuration

Tests/
├── Unit/                   # Unit tests (CI-friendly)
└── UI/                     # UI tests (local only)

docs/                       # Comprehensive documentation
scripts/                    # Build and development scripts
```

## Key Services & Components

### Networking Layer
- **ProxyClient**: Routes requests through calry.org proxy service
- **FoodDataClient**: Main API client interface
- **FoodDataCachedClient**: Caching layer with MainActor isolation
- **APIConfiguration**: Centralized API configuration management

### Data Models
- **BarcodeLookupResult**: Union type for FDC/OFF barcode lookups
- **FdcEnvelope/OffEnvelope**: Type aliases for envelope wrapping
- **NormalizedFood**: Canonical food data model
- **FoodMinimalCard**: UI display model

### Core Services
- **FoodNormalizationService**: Data normalization and merging
- **FDCCacheService**: @MainActor caching service
- **FoodDataConverter**: Model conversion utilities

## Development Standards

### Swift 6 Concurrency Compliance
- **MainActor Isolation**: Use `MainActor.run` for cross-actor access
- **Actor Boundaries**: Proper isolation for UI-related code
- **Sendable Conformance**: Avoid `@unchecked Sendable` when possible
- **Thread Safety**: Ensure all cache operations are properly isolated

### Code Quality Tools
- **SwiftLint**: Code style enforcement (0 violations target)
- **SwiftFormat**: Automatic code formatting with conflict resolution
- **Custom Rules**: Project-specific linting rules
- **CI Integration**: Automated code quality checks

### Error Handling
- **ProxyError**: Comprehensive error types with user-friendly messages
- **FoodDataError**: Custom error cases including `customError(String)`
- **Error Semantics**: Preserve specific error messages from proxy
- **Recovery Suggestions**: Provide actionable error recovery guidance

## API Integration Patterns

### Barcode Lookup Architecture
- **Union Type Design**: `BarcodeLookupResult` supports both FDC and OFF data
- **FDC Redirects**: Return actual FDC data (`.fdc` case)
- **OFF Redirects**: Return actual OFF data (`.off` case)
- **Data Preservation**: Maintain original data source and quality

### Configuration Management
- **APIConfiguration Service**: Reads from `.xcconfig` and `Info.plist`
- **Split URL Components**: Scheme, host, and path for flexible configuration
- **Environment Support**: Debug vs Release configurations
- **Type Safety**: Compile-time configuration validation

## Local Development Environment

### CI Environment Replication
- **Xcode 26.0.0**: Exact version matching CI
- **iOS 26 Runtime**: iPhone 16 simulator
- **Swift 6 Strict Concurrency**: Complete checking enabled
- **Build Scripts**: Automated setup, build, test, and lint scripts

### Available Scripts
- `setup-local-ci.sh`: Environment setup and verification
- `build-local-ci.sh`: CI-equivalent build with strict concurrency
- `test-local-ci.sh`: CI-equivalent test execution
- `lint-local-ci.sh`: Code quality checks

### Environment Variables
```bash
export CI_OFFLINE_MODE=YES
export NETWORK_TESTING_DISABLED=YES
export ENABLE_PREVIEWS=NO
export SWIFT_STRICT_CONCURRENCY=complete
export OTHER_SWIFT_FLAGS='-warnings-as-errors'
```

### iOS 26 Development Resources
- **Xcode Intelligence Chat Resources**: `/Applications/Xcode.app/Contents/PlugIns/IDEIntelligenceChat.framework/Versions/A/Resources`
- **Usage**: Reference these resources when implementing iOS 26 specific features
- **API Documentation**: Contains latest iOS 26 APIs, frameworks, and development patterns
- **Troubleshooting**: Use for iOS 26 compatibility issues and new feature implementation

## Documentation Standards

### Comprehensive Documentation
- **Architecture**: System design and structure documentation
- **API Integration**: Complete data flow and integration guides
- **Development**: Local setup, coding standards, testing, debugging
- **CI/CD**: Build process, test strategy, troubleshooting
- **Changelog**: Organized version history with detailed change tracking

### Documentation Structure
- **Tree Maps**: Visual project organization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tyson-hu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
