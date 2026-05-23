---
trigger: always_on
description: Guidelines for implementing platform-specific code for Windows, macOS, and Linux
---


# Platform-Specific Implementation Rules

All platform-specific code lives in [src/platform/](mdc:src/platform) organized by operating system. Each platform implements the same cross-platform interfaces using native APIs.

## Platform Directories

```
src/platform/
├── windows/    # Windows implementation (*.cpp)
├── macos/      # macOS implementation (*.mm for Objective-C++)
└── linux/      # Linux implementation (*.cpp with GTK)
```

## Platform APIs Used

| Platform | Primary APIs | Language | File Extension |
|----------|-------------|----------|----------------|
| **Windows** | Win32 API, GDI+ | C++ | `.cpp` |
| **macOS** | Cocoa/AppKit | Objective-C++ | `.mm` |
| **Linux** | GTK 3.0, X11 | C++ | `.cpp` |

## File Naming Convention

Platform files follow: `<module>_<platform>.<ext>`

Examples:
- `window_windows.cpp` - Windows window implementation
- `window_macos.mm` - macOS window implementation  
- `window_linux.cpp` - Linux window implementation
- `menu_windows.cpp` - Windows menu implementation

## Implementation Pattern

### 1. Define Platform-Specific PIMPL::Impl

Each platform file defines the `Impl` class declared in the cross-platform header:

#### Windows Example ([platform/windows/window_windows.cpp](mdc:src/platform/windows))

```cpp
// clang-format off
#include <windows.h>
#include <shellapi.h>
// clang-format on
#include "../../window.h"

namespace nativeapi {

// Define platform-specific Impl
class Window::Impl {
public:
    Impl(HWND hwnd) : hwnd_(hwnd) {}
    
    HWND hwnd_;  // Windows window handle
    // Other Windows-specific state...
};

// Implement constructors
Window::Window() : pimpl_(std::make_unique<Impl>(nullptr)) {}

Window::Window(void* window) 
    : pimpl_(std::make_unique<Impl>(static_cast<HWND>(window))) {}

Window::~Window() = default;

// Implement methods using Win32 API
void Window::Show() {
    if (pimpl_->hwnd_) {
        ShowWindow(pimpl_->hwnd_, SW_SHOW);
        SetForegroundWindow(pimpl_->hwnd_);
    }
}

void Window::Hide() {
    if (pimpl_->hwnd_) {
        ShowWindow(pimpl_->hwnd_, SW_HIDE);
    }
}

bool Window::IsVisible() const {
    return pimpl_->hwnd_ && IsWindowVisible(pimpl_->hwnd_);
}

void* Window::GetNativeObjectInternal() const {
    return static_cast<void*>(pimpl_->hwnd_);
}

}  // namespace nativeapi
```

#### macOS Example ([platform/macos/window_macos.mm](mdc:src/platform/macos))

```objc
#import <Cocoa/Cocoa.h>
#include "../../window.h"

namespace nativeapi {

// Define platform-specific Impl
class Window::Impl {
public:
    Impl(NSWindow* window) : window_(window) {
        if (window_) {
            [window_ retain];  // Retain ownership
        }
    }
    
    ~Impl() {
        if (window_) {
            [window_ release];  // Release ownership
        }
    }
    
    NSWindow* window_;  // macOS window handle
};

// Implement constructors
Window::Window() : pimpl_(std::make_unique<Impl>(nil)) {}

Window::Window(void* window) 
    : pimpl_(std::make_unique<Impl>(static_cast<NSWindow*>(window))) {}

Window::~Window() = default;

// Implement methods using Cocoa API
void Window::Show() {
    if (pimpl_->window_) {
        [pimpl_->window_ makeKeyAndOrderFront:nil];
    }
}

void Window::Hide() {
    if (pimpl_->window_) {
        [pimpl_->window_ orderOut:nil];
    }
}

bool Window::IsVisible() const {
    return pimpl_->window_ && [pimpl_->window_ isVisible];
}

void* Window::GetNativeObjectInternal() const {
    return static_cast<void*>(pimpl_->window_);
}

}  // namespace nativeapi
```

#### Linux Example ([platform/linux/window_linux.cpp](mdc:src/platform/linux))

```cpp
#include <gtk/gtk.h>
#include "../../window.h"

namespace nativeapi {

// Define platform-specific Impl
class Window::Impl {
public:
    Impl(GtkWidget* window) : window_(window) {
        if (window_) {
            g_object_ref(window_);  // Increase reference count
        }
    }
    
    ~Impl() {
        if (window_) {
            g_object_unref(window_);  // Decrease reference count
        }
    }
    
    GtkWidget* window_;  // GTK window handle
};

// Implement constructors
Window::Window() : pimpl_(std::make_unique<Impl>(nullptr)) {}

Window::Window(void* window) 
    : pimpl_(std::make_unique<Impl>(static_cast<GtkWidget*>(window))) {}

Window::~Window() = default;

// Implement methods using GTK API
void Window::Show() {
    if (pimpl_->window_) {
        gtk_widget_show_all(pimpl_->window_);
        gtk_window_present(GTK_WINDOW(pimpl_->window_));
    }
}

void Window::Hide() {
    if (pimpl_->window_) {
        gtk_widget_hide(pimpl_->window_);
    }
}

bool Window::IsVisible() const {
    return pimpl_->window_ && gtk_widget_get_visible(pimpl_->window_);
}

void* Window::GetNativeObjectInternal() const {
    return static_cast<void*>(pimpl_->window_);
}

}  // namespace nativeapi
```

## Platform Handle Types

### Native Handle Mapping

| Component | Windows | macOS | Linux |
|-----------|---------|-------|-------|
| Window | `HWND` | `NSWindow*` | `GtkWidget*` (GtkWindow) |
| Menu | `HMENU` | `NSMenu*` | `GtkWidget*` (GtkMenu) |
| MenuItem | N/A (in HMENU) | `NSMenuItem*` | `GtkWidget*` (GtkMenuItem) |
| Display | `HMONITOR` | `NSScreen*` | `GdkDisplay*` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libnativeapi/nativeapi](https://github.com/libnativeapi/nativeapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
