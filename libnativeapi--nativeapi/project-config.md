---
trigger: always_on
description: Project structure, organization, and architectural patterns
---


# Project Architecture Rules

This document describes the overall architecture and organization of the nativeapi project.

## Project Overview

**nativeapi** is a modern cross-platform C++ library providing unified access to native system APIs across Windows, macOS, and Linux. The library abstracts platform-specific details behind a clean, type-safe C++ interface and provides optional C bindings for FFI compatibility.

## Directory Structure

```
nativeapi/
├── include/              # Public API headers
│   └── nativeapi.h      # Single include file for all functionality
├── src/                  # Source code
│   ├── foundation/      # Core utilities (events, geometry, ID allocation)
│   ├── capi/           # C API bindings (FFI-friendly)
│   ├── platform/       # Platform-specific implementations
│   │   ├── windows/   # Windows implementations (*.cpp)
│   │   ├── macos/     # macOS implementations (*.mm)
│   │   └── linux/     # Linux implementations (*.cpp with GTK)
│   └── *.h, *.cpp     # Cross-platform interface definitions
├── examples/            # Example applications
└── docs/               # Documentation
```

## Core Architectural Layers

### 1. Foundation Layer ([src/foundation/](mdc:src/foundation))

Provides fundamental utilities used throughout the library:

- **[event.h](mdc:src/foundation/event.h)** - Base event class with timestamps
- **[event_emitter.h](mdc:src/foundation/event_emitter.h)** - Generic event system with listeners
- **[geometry.h](mdc:src/foundation/geometry.h)** - Cross-platform geometry types (Point, Size, Rectangle)
- **[id_allocator.h](mdc:src/foundation/id_allocator.h)** - Thread-safe ID generation
- **[native_object_provider.h](mdc:src/foundation/native_object_provider.h)** - Base class for exposing native handles

### 2. Cross-Platform Interface Layer ([src/](mdc:src))

Defines platform-agnostic APIs that all platforms must implement:

- **Window Management** - [window.h](mdc:src/window.h), [window_manager.h](mdc:src/window_manager.h)
- **Display Management** - [display.h](mdc:src/display.h), [display_manager.h](mdc:src/display_manager.h)
- **Tray Icons** - [tray_icon.h](mdc:src/tray_icon.h), [tray_manager.h](mdc:src/tray_manager.h)
- **Menus** - [menu.h](mdc:src/menu.h)
- **Keyboard Monitoring** - [keyboard_monitor.h](mdc:src/keyboard_monitor.h)
- **Accessibility** - [accessibility_manager.h](mdc:src/accessibility_manager.h)
- **Events** - [window_event.h](mdc:src/window_event.h), [display_event.h](mdc:src/display_event.h), etc.

### 3. Platform-Specific Implementation Layer

Each platform implements the cross-platform interfaces using native APIs:

- **Windows** - Uses Win32 API (HWND, HMENU, GDI+)
- **macOS** - Uses Cocoa/AppKit (NSWindow, NSMenu, Objective-C++)
- **Linux** - Uses GTK 3.0 (GtkWindow, GtkMenu)

### 4. C API Layer ([src/capi/](mdc:src/capi))

Provides C-compatible bindings for all C++ APIs to enable FFI from other languages:

- Each C++ API has a corresponding `_c.h` and `_c.cpp` file
- Uses opaque pointers and plain C types
- Memory management follows C conventions (explicit free functions)

## Key Design Patterns

### Singleton Pattern

Managers use Meyer's singleton pattern for global access:

```cpp
class WindowManager {
public:
    static WindowManager& GetInstance();
private:
    WindowManager();  // Private constructor
};
```

Examples: [WindowManager](mdc:src/window_manager.h), [DisplayManager](mdc:src/display_manager.h), [TrayManager](mdc:src/tray_manager.h)

### PIMPL (Pointer to Implementation)

All cross-platform classes use PIMPL to hide platform-specific details:

```cpp
class Window {
private:
    class Impl;  // Forward declaration
    std::unique_ptr<Impl> pimpl_;  // Platform-specific implementation
};
```

See [PIMPL Pattern Rules](mdc:.cursor/rules/pimpl-pattern.mdc) for details.

### Event-Driven Architecture

All managers and many objects inherit from `EventEmitter` to provide event notifications:

```cpp
class WindowManager : public EventEmitter<WindowEvent> {
    // Can emit WindowCreatedEvent, WindowClosedEvent, etc.
};
```

See [Event System Rules](mdc:.cursor/rules/event-system.mdc) for details.

### Native Object Provider

Classes that wrap platform-specific objects inherit from `NativeObjectProvider` to expose native handles:

```cpp
class Window : public NativeObjectProvider {
protected:
    void* GetNativeObjectInternal() const override;  // Returns HWND, NSWindow*, or GtkWidget*
};
```

## Build System

The project uses CMake ([CMakeLists.txt](mdc:CMakeLists.txt)) with platform detection:

- **C++17 Standard** required
- **Conditional Compilation** - Platform sources selected based on `CMAKE_SYSTEM_NAME`
- **Platform Dependencies**:
  - Windows: user32, shell32, dwmapi, gdiplus
  - macOS: Cocoa framework, Objective-C++ enabled
  - Linux: GTK 3.0, X11, ayatana-appindicator

## Naming Conventions

### C++ API

- **Classes**: PascalCase (e.g., `WindowManager`, `MenuItem`)
- **Methods**: PascalCase (e.g., `GetSize()`, `SetVisible()`)
- **Member Variables**: snake_case with trailing underscore (e.g., `window_id_`, `pimpl_`)
- **Enums**: PascalCase for type, PascalCase for values (e.g., `MenuItemType::Checkbox`)
- **Files**: snake_case (e.g., `window_manager.h`, `display_event.h`)

### C API


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libnativeapi/nativeapi](https://github.com/libnativeapi/nativeapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
