---
trigger: always_on
description: Pattern for exposing platform-specific native handles from cross-platform wrappers
---


# Native Object Provider Pattern Rules

Classes that wrap platform-specific objects inherit from [NativeObjectProvider](mdc:src/foundation/native_object_provider.h) to expose their underlying native handles. This enables advanced use cases where users need direct access to platform APIs while maintaining the cross-platform abstraction.

## Purpose

The NativeObjectProvider pattern serves several purposes:

1. **Escape Hatch** - Allows access to native APIs not wrapped by the library
2. **Interop** - Enables integration with other libraries expecting native handles
3. **Advanced Features** - Supports platform-specific functionality
4. **Type Safety** - Returns `void*` for cross-platform compatibility
5. **Encapsulation** - Keeps implementation details hidden until explicitly requested

## Base Class Structure

### Header ([foundation/native_object_provider.h](mdc:src/foundation/native_object_provider.h))

```cpp
#pragma once

namespace nativeapi {

class NativeObjectProvider {
public:
    virtual ~NativeObjectProvider() = default;
    
    /**
     * Get the native platform-specific object.
     * 
     * Platform-specific return types:
     * - macOS: NSWindow*, NSMenu*, NSMenuItem*, NSView*, etc.
     * - Windows: HWND, HMENU, etc.
     * - Linux: GtkWidget*, GtkMenu*, GdkWindow*, etc.
     */
    void* GetNativeObject() const {
        return GetNativeObjectInternal();
    }

protected:
    /**
     * Derived classes must implement this to return their native object.
     */
    virtual void* GetNativeObjectInternal() const = 0;
};

}  // namespace nativeapi
```

## Implementing NativeObjectProvider

### Pattern for Classes

All classes wrapping native objects should:

1. Inherit from `NativeObjectProvider`
2. Implement `GetNativeObjectInternal()` protected method
3. Return platform-specific handle as `void*`

### Example: Window Class

#### Header ([window.h](mdc:src/window.h))

```cpp
#pragma once
#include "foundation/native_object_provider.h"

namespace nativeapi {

class Window : public NativeObjectProvider {
public:
    Window();
    Window(void* native_window);
    virtual ~Window();
    
    // ... public API methods ...

protected:
    void* GetNativeObjectInternal() const override;

private:
    class Impl;
    std::unique_ptr<Impl> pimpl_;
};

}  // namespace nativeapi
```

#### Platform Implementations

##### Windows ([platform/windows/window_windows.cpp](mdc:src/platform/windows))

```cpp
#include <windows.h>
#include "../../window.h"

namespace nativeapi {

class Window::Impl {
public:
    HWND hwnd_;
};

void* Window::GetNativeObjectInternal() const {
    // Cast HWND to void*
    return static_cast<void*>(pimpl_->hwnd_);
}

}  // namespace nativeapi
```

##### macOS ([platform/macos/window_macos.mm](mdc:src/platform/macos))

```objc
#import <Cocoa/Cocoa.h>
#include "../../window.h"

namespace nativeapi {

class Window::Impl {
public:
    NSWindow* window_;
};

void* Window::GetNativeObjectInternal() const {
    // Cast NSWindow* to void*
    return static_cast<void*>(pimpl_->window_);
}

}  // namespace nativeapi
```

##### Linux ([platform/linux/window_linux.cpp](mdc:src/platform/linux))

```cpp
#include <gtk/gtk.h>
#include "../../window.h"

namespace nativeapi {

class Window::Impl {
public:
    GtkWidget* window_;
};

void* Window::GetNativeObjectInternal() const {
    // Cast GtkWidget* to void*
    return static_cast<void*>(pimpl_->window_);
}

}  // namespace nativeapi
```

## Using Native Objects

### Cross-Platform Usage

Users can access native objects when they need platform-specific functionality. Since platform-specific implementations are separated into `/platform/{windows|macos|linux}` directories, users should cast the native handle to the appropriate platform-specific type:

```cpp
#include <nativeapi.h>

using namespace nativeapi;

auto& manager = WindowManager::GetInstance();
auto window = manager.Create(options);

// Get native handle
void* native = window->GetNativeObject();

// Cast to platform-specific type
// Windows: HWND
// macOS: NSWindow*
// Linux: GtkWidget* (GtkWindow)
```

### Platform-Specific Usage Examples

#### Windows Implementation
```cpp
// In Windows-specific code
HWND hwnd = static_cast<HWND>(window->GetNativeObject());
SetWindowLongPtr(hwnd, GWL_EXSTYLE, WS_EX_LAYERED);
```

#### macOS Implementation
```objc
// In macOS-specific code
NSWindow* nswindow = static_cast<NSWindow*>(window->GetNativeObject());
[nswindow setCollectionBehavior:NSWindowCollectionBehaviorFullScreenPrimary];
```

#### Linux Implementation
```cpp
// In Linux-specific code
GtkWidget* gtkwindow = static_cast<GtkWidget*>(window->GetNativeObject());
gtk_window_set_keep_above(GTK_WINDOW(gtkwindow), TRUE);
```

## Classes Using NativeObjectProvider

The following classes inherit from NativeObjectProvider:

| Class | Native Type (Windows) | Native Type (macOS) | Native Type (Linux) |
|-------|----------------------|---------------------|---------------------|
| [Window](mdc:src/window.h) | `HWND` | `NSWindow*` | `GtkWidget*` (GtkWindow) |
| [Display](mdc:src/display.h) | `HMONITOR` | `NSScreen*` | `GdkDisplay*` / `GdkMonitor*` |
| [Menu](mdc:src/menu.h) | `HMENU` | `NSMenu*` | `GtkWidget*` (GtkMenu) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libnativeapi/nativeapi](https://github.com/libnativeapi/nativeapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
