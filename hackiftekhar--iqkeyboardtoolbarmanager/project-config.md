---
trigger: always_on
description: IQKeyboardToolbarManager is an iOS Swift library that automatically manages keyboards and toolbars for text input views. It provides Previous/Next buttons and customizable toolbar functionality for iOS applications. The library supports both Swift and Objective-C integration and is distributed via CocoaPods, Swift Package Manager, and Carthage.
---

# IQKeyboardToolbarManager

IQKeyboardToolbarManager is an iOS Swift library that automatically manages keyboards and toolbars for text input views. It provides Previous/Next buttons and customizable toolbar functionality for iOS applications. The library supports both Swift and Objective-C integration and is distributed via CocoaPods, Swift Package Manager, and Carthage.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

- **CRITICAL - Network Limitations**: This development environment has limited network connectivity. CocoaPods dependency resolution and iOS simulator builds will fail. Focus on local validation and syntax checking.

- **Bootstrap and validate the repository**:
  - Install CocoaPods: `sudo gem install cocoapods` -- takes 30 seconds on first install. NEVER CANCEL.
  - Validate library structure: `pod lib lint --quick --skip-import-validation --allow-warnings` -- takes 0.5 seconds
  - Validate Swift syntax: `find . -name "*.swift" -exec swift -frontend -parse {} \;` -- takes 3 seconds  
  - Validate podspec: `pod spec lint --quick --skip-import-validation --allow-warnings` -- takes 0.5 seconds

- **Complete validation workflow** (run this after making changes):
  ```bash
  # Step 1: Syntax validation (3 seconds)
  find . -name "*.swift" -exec swift -frontend -parse {} \;
  
  # Step 2: Library structure validation (0.5 seconds)  
  pod lib lint --quick --skip-import-validation --allow-warnings
  
  # Step 3: Verify file structure
  ls -la IQKeyboardToolbarManager/Classes/
  ```

- **DO NOT attempt these commands** (they will fail due to environment limitations):
  - `pod install` in Example directory - fails due to network CDN access
  - `swift build` - fails because UIKit is not available on Linux  
  - Xcode builds or iOS simulator runs - not available on Linux
  - SwiftLint installation from source - takes 15+ minutes and may timeout

## Validation

- **Always run syntax validation** before making code changes:
  - Swift syntax: `find . -name "*.swift" -exec swift -frontend -parse {} \;`
  - Library validation: `pod lib lint --quick --skip-import-validation --allow-warnings`

- **Key file structure validation**:
  - Main library: `IQKeyboardToolbarManager/Classes/` contains Swift source files
  - Package definition: `Package.swift` (Swift Package Manager)
  - Pod specification: `IQKeyboardToolbarManager.podspec.json` 
  - Examples: `Example/IQKeyboardToolbarManager/` (Swift) and `Example/IQKeyboardToolbarManager_ObjcExample/` (Objective-C)

- **Always validate that changes maintain**:
  - iOS 13.0+ platform requirement
  - Swift 5.7+ compatibility  
  - Both Swift and Objective-C API compatibility
  - UIKit framework dependency structure

## Common Tasks

### Repository Structure
```
ls -la /
.github/                    # GitHub templates and configurations
IQKeyboardToolbarManager/   # Main library source code
├── Classes/               # Swift source files
│   ├── Configuration/     # IQKeyboardToolbarConfiguration.swift
│   ├── Constants/         # IQKeyboardToolbarConstants.swift
│   ├── Debug/            # Debug-related utilities
│   ├── Toolbar/          # Toolbar management logic
│   └── UIKitExtensions/  # UIKit extensions
└── Assets/               # Resources including PrivacyInfo.xcprivacy
Example/                  # Demo applications
├── IQKeyboardToolbarManager/        # Swift example app
├── IQKeyboardToolbarManager_ObjcExample/  # Objective-C example app  
└── Tests/                # Unit tests (minimal implementation)
Package.swift             # Swift Package Manager configuration
IQKeyboardToolbarManager.podspec.json  # CocoaPods specification
Cartfile                 # Carthage dependencies
```

### Key Dependencies
```
# Swift Package Manager (Package.swift)
dependencies: [
    "IQKeyboardToolbar" (from: "1.1.2")
    "IQTextInputViewNotification" (from: "1.0.8")
]

# CocoaPods (podspec.json) 
dependencies: {
    "IQKeyboardToolbar": [],
    "IQTextInputViewNotification": []
}

# Carthage (Cartfile)
github "hackiftekhar/IQKeyboardToolbar"
github "hackiftekhar/IQTextInputViewNotification"
```

### Current Status Check Commands
```bash
# Check installed tools
which pod && pod --version              # Should show CocoaPods 1.16.2+
which swift && swift --version          # Should show Swift 5.7+

# Quick library health check  
pod lib lint --quick --skip-import-validation --allow-warnings --silent

# Count source files
find . -name "*.swift" | wc -l          # Should show 17+ Swift files
find Example -name "*.m" -o -name "*.h" | wc -l  # Should show 7+ Obj-C files
```

### Main Library API Overview
```swift
// Enable keyboard toolbar management
IQKeyboardToolbarManager.shared.isEnabled = true

// Configuration options
IQKeyboardToolbarManager.shared.toolbarConfiguration.tintColor = UIColor.systemGreen

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hackiftekhar/IQKeyboardToolbarManager](https://github.com/hackiftekhar/IQKeyboardToolbarManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
