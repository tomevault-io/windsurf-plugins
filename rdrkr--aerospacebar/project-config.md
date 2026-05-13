---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

### Build System

This project uses **Swift Package Manager (SPM)** for library targets (Domain, Data, Presentation) and standard **Xcode
** for the main application target.

```bash
# Build commands using xcodebuild
xcodebuild -scheme AeroSpaceBar -configuration Debug build
xcodebuild -scheme AeroSpaceBar -configuration Release build

# Clean build artifacts
xcodebuild -scheme AeroSpaceBar clean

# Or use the automation scripts
./Scripts/build.sh                     # Build Release configuration
./Scripts/build.sh -c Debug            # Build Debug configuration
./Scripts/build.sh --clean             # Clean and build

# Code quality (SwiftFormat and SwiftLint must be installed)
swiftformat .
swiftlint --fix
```

## Code Quality Tools

- **SwiftFormat**: Code formatting with `.swiftformat` config (120 char line limit, 4-space indentation)
- **SwiftLint**: Strict linting with `.swiftlint.yaml` config (extensive opt-in rules, analyzer rules)
- Both tools are aligned to prevent conflicts and should always be run before committing

## Architecture Overview

AeroSpaceBar follows **MVVM Clean Architecture** principles with strict layer separation:

### Project Structure

```
├── Packages/                          # Swift Package Manager packages
│   ├── Domain/                        # Business Logic Layer (SPM package)
│   │   ├── Sources/Domain/
│   │   │   ├── Entities/             # Core business models, configuration, logging
│   │   │   ├── Gateways/             # Repository contracts/protocols
│   │   │   └── UseCases/             # Application business logic operations
│   │   └── Tests/DomainTests/
│   ├── Data/                          # Data Access Layer (SPM package)
│   │   ├── Sources/Data/
│   │   │   ├── Models/               # External data models (from AeroSpace CLI)
│   │   │   ├── Network/              # AeroSpaceCLIClient, IconCache
│   │   │   └── Repositories/         # Gateway implementations
│   │   └── Tests/DataTests/
│   └── Presentation/                  # UI Layer (SPM package)
│       ├── Sources/Presentation/
│       │   ├── ViewModels/           # MVVM ViewModels using Combine
│       │   └── Views/                # SwiftUI Views + Common components
│       └── Tests/PresentationTests/
├── AeroSpaceBar/                      # Main application target
│   ├── AeroSpaceBarApp.swift         # App entry point
│   ├── Info.plist                    # App customizations (version in Xcode project)
│   ├── AeroSpaceBar.entitlements     # App entitlements
│   └── Resources/                    # App resources
├── AeroSpaceBar.xcodeproj/            # Xcode project
├── Scripts/                           # Automation scripts for release management
├── appcast.xml                        # Sparkle appcast feed for software updates
├── CLAUDE.md                          # Claude Code development instructions
└── README.md                          # Project documentation
```

### Key Architectural Patterns

1. **Clean Architecture**: Strict separation of concerns, dependencies point inward
2. **MVVM with Combine**: ViewModels use Combine for reactive data binding
3. **Functional Programming**: Favor immutability, pure functions, and higher-order functions
4. **Dependency Injection**: All dependencies managed through `DependencyContainer.swift`
5. **Use Case Pattern**: Each business operation isolated in its own use case class. Repositories' and gateways'
   functionality are only accessible via Use Cases
6. **Protocol-Oriented Design**: All gateways defined as protocols in Domain layer
7. **Repository Pattern**: Data access abstracted through gateway protocols
8. **Single Responsibility Principle**: Each class has one reason to change
9. **Separation of Concerns**: Clear boundaries between layers, no direct dependencies from
10. **Modern Swift**: Use the most modern (6.2+) language features and conventions. Target Swift 6 and use Swift
    concurrency (async/await, actors) and Swift macros where applicable.

### Core Components

- **DependencyContainer**: Centralized DI container managing all service lifetimes
- **SpacesGateway**: Main interface for AeroSpace window manager interaction
- **ConfigurationGateway**: User settings and preferences management
- **IconCache**: Performance-optimized application icon caching
- **AeroSpaceCLIClient**: Direct interface to AeroSpace CLI commands

## Development Patterns

### Adding New Features

1. Create domain entities and use cases first (no dependencies)
2. Add gateway protocols to Domain/Gateways/
3. Implement repositories in Data layer
4. Create ViewModels in Presentation/ViewModels/
5. Add Views in Presentation/Views/
6. Wire dependencies in DependencyContainer
7. Write tests for each layer

### Testing Strategy

- For every change you make, verify your changes by building: `./Scripts/build.sh -c Debug`
- **Unit Tests**:
  - `Packages/Domain/Tests/DomainTests/` - Domain layer testing
  - `Packages/Data/Tests/DataTests/` - Data layer testing
  - `Packages/Presentation/Tests/PresentationTests/` - Presentation layer testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rdrkr/AeroSpaceBar](https://github.com/rdrkr/AeroSpaceBar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
