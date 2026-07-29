---
trigger: always_on
description: **Target:** C++17 cross-platform IPC middleware
---

# Areg SDK — AI Agent Coding Rules

**Target:** C++17 cross-platform IPC middleware
**Audience:** AI agents generating or refactoring Areg SDK code

This document defines mandatory rules, recommendations, and prohibitions.
When in doubt, match the style of the surrounding code. Do not introduce new style variants.

**Companion document:** `./docs/AREG_CODING_STYLE.md` (developer-oriented style guide with rationale)

---

## 1. Naming Rules (MUST)

Every identifier in generated code must follow this table exactly.

| Element                        | Convention          | Example                            |
|--------------------------------|---------------------|------------------------------------|
| Class / Struct / Alias         | `PascalCase`        | `ServiceManager`, `ConnectionData` |
| Enum class                     | `PascalCase`        | `ConnectionState`                  |
| Enum values                    | `PascalCase`        | `ConnectionState::Connected`       |
| Functions / Methods            | `snake_case`        | `start_service()`                  |
| Class data members             | `mPascalCase`       | `mSocketFd`, `mAddress`            |
| Struct public fields           | `snake_case`        | `service_address`, `timeout_ms`    |
| Static class members           | `s_snake_case`      | `s_instance_count`                 |
| Static local variables         | `_snake_case`       | `_instance`                        |
| Global vars (internal linkage) | `_snake_case`       | `_max_connections`                 |
| Constants (`constexpr`)        | `UPPER_SNAKE_CASE`  | `MAX_BUFFER_SIZE`                  |
| Template parameters            | `PascalCase`        | `MessageType`                      |
| Namespaces                     | `snake_case`        | `areg`, `aregext`, `areg::os`      |
| Macros                         | `AREG_UPPER_SNAKE`  | `AREG_NOCOPY`                      |
| Local variables                | `snake_case`        | `connection_count`                 |

**Note:** The above table is a strict requirement for generated new code. Tolerate existing `camelCase` local variable style when reviewing. 

### 1.1 Accessor naming

| Pattern            | Purpose        | Example                             |
|--------------------|----------------|-------------------------------------|
| `property()`       | Accessor       | `size()`, `count()`, `state()`      |
| `is_property()`    | Boolean query  | `is_valid()`, `is_connected()`      |
| `has_property()`   | Boolean query  | `has_data()`, `has_parent()`        |
| `set_property()`   | Mutator        | `set_size()`, `set_name()`          |
| `action_noun()`    | Command        | `start_service()`, `stop_timer()`   |

- No `get_` prefix: write `count()`, not `get_count()`.
- Boolean accessors keep `is_` or `has_` prefix.
- Mutators always use `set_` prefix.

### 1.2 Name length

Prefer shorter names when context makes meaning clear.

```cpp
// Good — function provides context
void load_config(std::string_view file);
void set_timeout(int ms);

// Bad — too long
void load_configuration_file(std::string_view configuration_file_path);
```

### 1.3 Namespace assignment

Assign namespaces based on module location. Do not invent new top-level names.

| Namespace  | Module / location                                                        |
|------------|--------------------------------------------------------------------------|
| `areg`     | All files in `framework/areg/` (core framework)                          |
| `areg::os` | OS-specific files (suffix `Posix` or `Win32`) within `framework/areg/`  |
| `aregext`  | All files in `framework/aregextend/`                                     |
| `areglog`  | All files in `framework/areglogger/`                                     |

Standalone applications (`logcollector`, `logobserver`, `mtrouter`) do not require a namespace. If one is used, it must match the module name.

Wrap every `.hpp` and `.cpp` file in the correct namespace:

```cpp
// framework/areg/ — core framework
namespace areg
{
    class ServiceManager { };
}  // namespace areg

// framework/areg/ private/ posix/ or win32/ — OS-specific
namespace areg::os
{
    class TimerPosix { };
}  // namespace areg::os

// framework/aregextend/
namespace aregext
{
    class SystemServiceConsole { };
}  // namespace aregext
```

---

## 2. Type Rules (MUST)

### 2.1 String parameters

Use `std::string_view` for read-only string parameters.
Use `const char*` only at C API boundaries (`extern "C"`).

```cpp
void load_config(std::string_view file);            // Correct
void load_config(const char* file);                 // Wrong
```

### 2.2 Integer types

Use fixed-width types: `uint8_t`, `uint16_t`, `uint32_t`, `uint64_t`, `int8_t`, `int16_t`, `int32_t`, `int64_t`.
Never use `unsigned int`, `unsigned short`, or `unsigned char`.

**Exceptions:** `size_t` for STL compatibility; `int` for short loop counters.

### 2.3 Pointers and references

Attach `*` and `&` to the type. A space between type and `*`/`&` is also acceptable.

```cpp
const String& name;                                 // Correct
const String & name;                                // Also acceptable
const String &name;                                 // Wrong
```

### 2.4 nullptr


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aregtech/areg-sdk](https://github.com/aregtech/areg-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
