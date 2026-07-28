---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Use @agent-swift-cpp-interop.

## Project Overview

QwiftUI is an experimental Swift UI library based on Qt6, implementing cross-platform GUI development using Swift 6.2's C++ interoperability features. The library itself is standalone and does not depend on SwiftCrossUI.

**Key Goals**:

- Provide Qt6 access through direct C++ interop without C API bridging, leveraging Swift 6.2's enhanced C++ support
- Maintain QwiftUI as a standalone library for direct Qt6 access from Swift
- Provide optional SwiftCrossUI integration through a separate Qt6AppBackend target

## Build Commands

```bash
swift build                    # Build the project
swift test                     # Run tests (no tests currently)
swift package resolve          # Resolve dependencies
swift run QtDemo #runs the demo app
swift run SimpleTestDemo         # Run the UI testing app
```

## Architecture

The project has transitioned from a C wrapper approach to direct Swift C++ interoperability:

### Current Target Structure

1. **QtBridge** - C++ wrapper classes for Qt types
   - Location: `Sources/QtBridge/`
   - Purpose: Thin C++ layer providing Swift-compatible Qt widget wrappers
   - Key classes: `SwiftQApplication`, `SwiftQWidget`, `SwiftQLabel`
   - Uses standard C++ types (std::string) for Swift compatibility
   - Dependencies: None (pure C++)

2. **QwiftUI** - Standalone Swift API layer
   - Location: `Sources/QwiftUI/`
   - Purpose: High-level Swift wrappers around QtBridge C++ classes, think UIKit-style API around Qt6 for Swift
   - Key files:
     - `SimpleApp.swift` - Manages Qt application lifecycle
     - `Widget.swift` - Base widget wrapper, other controls can inherit from it
     - `Label.swift` - Label widget implementation
     - `Application.swift` - Application management
   - Dependencies: QtBridge only (no SwiftCrossUI dependency)

3. **Qt6AppBackend** - SwiftCrossUI integration (planned)
   - Location: `Sources/Qt6AppBackend/`
   - Purpose: Implements SwiftCrossUI's AppBackend protocol using QwiftUI
   - Dependencies: QwiftUI + SwiftCrossUI
   - Key components:
     - `QtBackend.swift` - AppBackend protocol implementation
     - `QtWindow.swift` - Window abstraction for SwiftCrossUI
     - `QtBackendWidget.swift` - Widget wrapper for AppBackend

4. **QtDemo** - Example executable
   - Location: `Sources/QtDemo/`
   - Purpose: Demonstrates direct C++ interop usage
   - Shows window creation, widget hierarchy, and event handling
   - Can optionally use Qt6AppBackend for SwiftCrossUI demos

### Legacy Target (Commented Out)

- **CQtWrapper** - Original C API wrapper (deprecated in favor of C++ interop)

## Swift C++ Interop Implementation

### Key Technical Details

- Swift version: 6.2 (required for enhanced C++ interop features)
- Interoperability mode: `.interoperabilityMode(.Cxx)` enabled on all targets
- Qt integration: Direct framework linking without module maps
- Memory management: Raw pointers instead of std::unique_ptr (avoids incomplete type issues)

### Current Implementation Status

**Working**:

- Direct C++ class instantiation from Swift
- Basic widget creation (QWidget, QLabel)
- Window properties (title, size, position)
- Factory functions for widget creation

**Known Issues**:

- QApplication must be created before any QWidget
- Type casting between widget classes needs refinement
- Qt namespace constants not directly accessible in Swift

### C++ Wrapper Design

```cpp
// Example from QtBridge.h
class SwiftQApplication {
private:
    QApplication* app;
public:
    SwiftQApplication(int& argc, char** argv);
    ~SwiftQApplication();
    int exec();
};
```

## Platform-Specific Setup

**macOS (Homebrew)**:

- Qt6 installed via: `brew install qt`
- Expected location: `/opt/homebrew/Cellar/qt/6.9.1/`
- Frameworks linked: QtCore, QtWidgets, QtGui

**Linux**:

- Qt6 headers expected at: `/usr/include/qt6`
- System package installation required

**Note**: Qt paths are hardcoded in Package.swift and need updating for different Qt versions.

## Development Workflow

### Adding New Qt Widget Support

1. Add C++ wrapper class to `QtBridge.h`:

   ```cpp
   class SwiftQButton : public SwiftQWidget {
   public:
       SwiftQButton(const std::string& text);
       void setText(const std::string& text);
   };
   ```

2. Implement in `QtBridge.cpp`

3. Create Swift wrapper in QwiftUI target

4. Use factory functions for complex object creation to avoid initialization order issues

### Updating Qt Version

1. Update paths in Package.swift:
   - `cxxSettings` include paths
   - `linkerSettings` framework paths
2. Ensure Qt version compatibility with C++ features used

### Debugging Tips

- Check QApplication initialization order when encountering "Must construct a QApplication before a QWidget" errors
- Use factory functions (`createWidget`, `createLabel`) to ensure proper C++ object construction
- Print statements help trace initialization order issues
- Qt applications require running event loop, it must always be started for executable targets.

## Project Constraints

1. **Swift Package Manager only** - No additional build systems

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kkostov/QwiftUI](https://github.com/kkostov/QwiftUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
