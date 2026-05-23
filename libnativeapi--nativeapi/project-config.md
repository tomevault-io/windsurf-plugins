---
trigger: always_on
description: Singleton manager pattern for system-wide resource management
---


# Singleton Manager Pattern Rules

System-wide resources (windows, displays, tray icons, keyboard monitoring) are managed by singleton manager classes. This ensures consistent state management and centralized event emission across the application.

## Manager Classes

The following managers use the singleton pattern:

- **[WindowManager](mdc:src/window_manager.h)** - Manages all application windows
- **[DisplayManager](mdc:src/display_manager.h)** - Manages display/monitor information
- **[TrayManager](mdc:src/tray_manager.h)** - Manages system tray icons
- **[AccessibilityManager](mdc:src/accessibility_manager.h)** - Manages accessibility permissions

## Singleton Pattern Structure

### Meyer's Singleton Pattern

All managers use Meyer's singleton (thread-safe in C++11+):

```cpp
class WindowManager : public EventEmitter<WindowEvent> {
public:
    // Get singleton instance
    static WindowManager& GetInstance() {
        static WindowManager instance;  // Created on first call
        return instance;
    }

    virtual ~WindowManager();

    // Prevent copying and moving
    WindowManager(const WindowManager&) = delete;
    WindowManager& operator=(const WindowManager&) = delete;
    WindowManager(WindowManager&&) = delete;
    WindowManager& operator=(WindowManager&&) = delete;

private:
    // Private constructor
    WindowManager();
};
```

**Key Points:**
- Static local variable ensures single instance
- Thread-safe initialization (C++11 guarantee)
- Private constructor prevents direct instantiation
- Deleted copy/move prevents duplication
- Returns reference (not pointer) to prevent deletion

## Complete Manager Template

### Header File Pattern ([window_manager.h](mdc:src/window_manager.h))

```cpp
#pragma once
#include <memory>
#include <vector>
#include <unordered_map>
#include "foundation/event_emitter.h"
#include "window.h"
#include "window_event.h"

namespace nativeapi {

class WindowManager : public EventEmitter<WindowEvent> {
public:
    // Singleton access
    static WindowManager& GetInstance();

    virtual ~WindowManager();

    // Public API
    std::shared_ptr<Window> Create(const WindowOptions& options);
    std::shared_ptr<Window> Get(WindowId id);
    std::vector<std::shared_ptr<Window>> GetAll();
    bool Destroy(WindowId id);

    // Prevent copying and moving
    WindowManager(const WindowManager&) = delete;
    WindowManager& operator=(const WindowManager&) = delete;
    WindowManager(WindowManager&&) = delete;
    WindowManager& operator=(WindowManager&&) = delete;

private:
    // Private constructor
    WindowManager();

    // PIMPL for platform-specific details
    class Impl;
    std::unique_ptr<Impl> pimpl_;

    // Shared state (not platform-specific)
    std::unordered_map<WindowId, std::shared_ptr<Window>> windows_;

    // Platform event monitoring
    void SetupEventMonitoring();
    void CleanupEventMonitoring();
    void DispatchWindowEvent(const WindowEvent& event);
};

}  // namespace nativeapi
```

### Implementation Pattern ([window_manager.cpp](mdc:src/window_manager.cpp))

```cpp
#include "window_manager.h"

namespace nativeapi {

WindowManager& WindowManager::GetInstance() {
    static WindowManager instance;
    return instance;
}

WindowManager::WindowManager() : pimpl_(std::make_unique<Impl>(this)) {
    SetupEventMonitoring();
}

WindowManager::~WindowManager() {
    CleanupEventMonitoring();
}

std::shared_ptr<Window> WindowManager::Create(const WindowOptions& options) {
    // Platform-specific creation
    auto window = pimpl_->CreatePlatformWindow(options);

    if (window) {
        // Store in registry
        windows_[window->GetId()] = window;

        // Emit event
        Emit<WindowCreatedEvent>(window->GetId());
    }

    return window;
}

std::shared_ptr<Window> WindowManager::Get(WindowId id) {
    auto it = windows_.find(id);
    return (it != windows_.end()) ? it->second : nullptr;
}

std::vector<std::shared_ptr<Window>> WindowManager::GetAll() {
    std::vector<std::shared_ptr<Window>> result;
    result.reserve(windows_.size());

    for (const auto& [id, window] : windows_) {
        result.push_back(window);
    }

    return result;
}

bool WindowManager::Destroy(WindowId id) {
    auto it = windows_.find(id);
    if (it == windows_.end()) {
        return false;
    }

    // Platform-specific cleanup happens in Window destructor
    windows_.erase(it);

    // Emit event
    Emit<WindowClosedEvent>(id);

    return true;
}

}  // namespace nativeapi
```

## Usage Patterns

### Pattern 1: Accessing the Singleton

```cpp
// Get reference to manager
auto& manager = WindowManager::GetInstance();

// Use manager
auto window = manager.Create(options);
```

**Never:**
```cpp
// Don't create pointers to singleton
WindowManager* manager = &WindowManager::GetInstance();  // Unnecessary

// Don't try to create instances
WindowManager manager;  // Won't compile - private constructor
```

### Pattern 2: Registering Event Listeners

Managers inherit from `EventEmitter`, so you can add listeners:

```cpp
auto& manager = WindowManager::GetInstance();

// Register listener for specific event
auto listener_id = manager.AddListener<WindowCreatedEvent>(
    [](const WindowCreatedEvent& event) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libnativeapi/nativeapi](https://github.com/libnativeapi/nativeapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
