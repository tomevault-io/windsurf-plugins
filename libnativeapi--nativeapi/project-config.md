---
trigger: always_on
description: Creating C API bindings for C++ APIs to enable FFI from other languages
---


# C API Bindings Rules

The nativeapi library provides C-compatible bindings for all C++ APIs to enable Foreign Function Interface (FFI) from languages like Dart, Swift, Rust, Python, etc. All C API code lives in [src/capi/](mdc:src/capi).

## Why C API Bindings?

1. **Language Interoperability** - C is the universal FFI standard
2. **Stable ABI** - C has predictable memory layout and calling conventions
3. **No Name Mangling** - C functions have simple, predictable names
4. **Simplicity** - C types map directly to FFI types in most languages

## File Organization

For each C++ API, create corresponding C binding files:

| C++ API | C API Header | C API Implementation |
|---------|-------------|---------------------|
| [window.h](mdc:src/window.h) | [window_c.h](mdc:src/capi/window_c.h) | [window_c.cpp](mdc:src/capi/window_c.cpp) |
| [window_manager.h](mdc:src/window_manager.h) | [window_manager_c.h](mdc:src/capi/window_manager_c.h) | [window_manager_c.cpp](mdc:src/capi/window_manager_c.cpp) |
| [menu.h](mdc:src/menu.h) | [menu_c.h](mdc:src/capi/menu_c.h) | [menu_c.cpp](mdc:src/capi/menu_c.cpp) |

## C API Header Pattern

### Basic Structure ([window_c.h](mdc:src/capi/window_c.h))

```c
#pragma once

#include <stdbool.h>
#include <stdint.h>

// Export macro for DLL support
#if _WIN32
#define FFI_PLUGIN_EXPORT __declspec(dllexport)
#else
#define FFI_PLUGIN_EXPORT
#endif

#ifdef __cplusplus
extern "C" {
#endif

#include "geometry_c.h"  // Include C versions of dependencies

// Opaque handle type
typedef void* native_window_t;
typedef long native_window_id_t;

// C struct for options (plain C types only)
typedef struct {
    const char* title;
    native_size_t size;
    native_size_t minimum_size;
    native_size_t maximum_size;
    bool centered;
} native_window_options_t;

// C functions matching C++ API
FFI_PLUGIN_EXPORT
native_window_t native_window_create(const native_window_options_t* options);

FFI_PLUGIN_EXPORT
void native_window_destroy(native_window_t window);

FFI_PLUGIN_EXPORT
native_window_id_t native_window_get_id(native_window_t window);

FFI_PLUGIN_EXPORT
void native_window_show(native_window_t window);

FFI_PLUGIN_EXPORT
void native_window_hide(native_window_t window);

FFI_PLUGIN_EXPORT
bool native_window_is_visible(native_window_t window);

// Memory management helpers
FFI_PLUGIN_EXPORT
void native_window_free(native_window_t window);

#ifdef __cplusplus
}
#endif
```

### Key Elements

1. **Export Macro** - `FFI_PLUGIN_EXPORT` for DLL/shared library support
2. **Extern "C" Block** - Prevents C++ name mangling
3. **Opaque Pointers** - `typedef void* native_xxx_t` for object handles
4. **Plain C Types** - Only `bool`, `int`, `float`, `double`, `char*`, structs
5. **Naming Convention** - `native_<module>_<function>` pattern
6. **Documentation** - Comment each function for FFI consumers

## C API Implementation Pattern

### Basic Structure ([window_c.cpp](mdc:src/capi/window_c.cpp))

```cpp
#include "window_c.h"
#include <memory>
#include "../window.h"
#include "../window_manager.h"
#include "string_utils_c.h"

using namespace nativeapi;

// Convert C options to C++ options
static WindowOptions ConvertToWindowOptions(const native_window_options_t* options) {
    WindowOptions cpp_options;
    
    if (options->title) {
        cpp_options.title = std::string(options->title);
    }
    
    cpp_options.size.width = options->size.width;
    cpp_options.size.height = options->size.height;
    cpp_options.minimum_size.width = options->minimum_size.width;
    cpp_options.minimum_size.height = options->minimum_size.height;
    cpp_options.maximum_size.width = options->maximum_size.width;
    cpp_options.maximum_size.height = options->maximum_size.height;
    cpp_options.centered = options->centered;
    
    return cpp_options;
}

// Convert C++ window to C handle
static native_window_t WindowToHandle(std::shared_ptr<Window> window) {
    return window ? static_cast<void*>(window.get()) : nullptr;
}

// Convert C handle to C++ window
static std::shared_ptr<Window> HandleToWindow(native_window_t handle) {
    if (!handle) return nullptr;
    
    // Get from manager's internal registry
    Window* raw_ptr = static_cast<Window*>(handle);
    return WindowManager::GetInstance().Get(raw_ptr->GetId());
}

// Implement C functions
native_window_t native_window_create(const native_window_options_t* options) {
    if (!options) return nullptr;
    
    try {
        auto cpp_options = ConvertToWindowOptions(options);
        auto window = WindowManager::GetInstance().Create(cpp_options);
        return WindowToHandle(window);
    } catch (...) {
        return nullptr;
    }
}

void native_window_destroy(native_window_t window) {
    try {
        auto cpp_window = HandleToWindow(window);
        if (cpp_window) {
            WindowManager::GetInstance().Destroy(cpp_window->GetId());
        }
    } catch (...) {
        // Ignore exceptions
    }
}

native_window_id_t native_window_get_id(native_window_t window) {
    try {
        auto cpp_window = HandleToWindow(window);
        return cpp_window ? cpp_window->GetId() : 0;
    } catch (...) {
        return 0;
    }
}

void native_window_show(native_window_t window) {
    try {
        auto cpp_window = HandleToWindow(window);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libnativeapi/nativeapi](https://github.com/libnativeapi/nativeapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
