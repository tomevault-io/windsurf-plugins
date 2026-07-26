---
trigger: always_on
description: ReGenny is a reverse engineering GUI tool for interactively reconstructing memory structures in live Windows processes. It parses `.genny` files (C-like struct definitions), attaches to a target process, and renders live memory as typed structures.
---

# ReGenny Agent Guide

ReGenny is a reverse engineering GUI tool for interactively reconstructing memory structures in live Windows processes. It parses `.genny` files (C-like struct definitions), attaches to a target process, and renders live memory as typed structures.

Full scripting reference: https://praydog.github.io/regenny-book/

## Quick Start Workflow

1. **Check status**: `regenny_status` -- see if a file is open and a process is attached
2. **List processes**: `regenny_list_processes` -- find the target process
3. **Attach**: `regenny_attach` with PID or process name
4. **Open/create a .genny file**: `regenny_open_file` or `regenny_new_file`
5. **List types**: `regenny_list_types` -- see what structs are defined
6. **Select a type**: `regenny_select_type` with a name and address to view it in the UI
7. **Read memory**: `regenny_read_memory` or `regenny_read_typed` for raw access
8. **Modify definitions**: `regenny_set_file` to update the .genny content (triggers re-parse)
9. **Use Lua**: `regenny_lua_eval` for complex operations using the full scripting API

## Address Syntax

The address field (and `regenny_select_type` address parameter) supports:

- **Absolute hex**: `0x7FF6A000`
- **Module-relative**: `game.exe+0x5000` (partial module name OK, e.g. `client.dll+0x1234`)
- **Pointer dereference chains**: `game.exe+0x5000->0x24->0x8` (follows pointers at each `->` offset)

## The .genny File Format

`.genny` files define types using a C-like syntax. They are parsed by ReGenny and used to overlay memory.

### Primitive Types

Primitives must be declared with a name, byte size, and optional metadata tag:

```
type bool 1 [[bool]]
type byte 1 [[u8]]
type ubyte 1 [[u8]]
type char 1 [[utf8]]
type wchar_t 2 [[utf16]]
type short 2 [[i16]]
type ushort 2 [[u16]]
type int 4 [[i32]]
type uint 4 [[u32]]
type float 4 [[f32]]
type double 8 [[f64]]
type int64_t 8 [[i64]]
type uint64_t 8 [[u64]]
type uintptr_t 8 [[u64]]
```

Metadata tags control how values are displayed: `[[i32]]` = signed 32-bit int, `[[f32]]` = float, `[[utf8*]]` = null-terminated string pointer, etc. Metadata can also be applied per-variable inline: `char* name [[utf8*]]`.

### Structs

```
struct MyStruct {
    int field_a
    float field_b
    bool flag
}
```

`class` is also supported (identical semantics to `struct` in .genny).

### Explicit Offsets

Use `+N` to place a field at a specific byte offset:

```
struct Player {
    int health +0x10
    int max_health
    float position_x +0x40
    float position_y
    float position_z
}
```

Fields without `+N` are placed sequentially after the previous field.

### Explicit Struct Size

```
struct PaddedStruct 0x100 {
    int value
}
```

This declares the struct as 0x100 bytes total, regardless of field layout.

### Pointers

```
struct Node {
    int value
    Node* next
    char* name [[utf8*]]
}
```

### Arrays

```
struct Container {
    int values[10]
    float matrix[4][4]
    Item* items
}
```

### Enums

```
enum Direction {
    NORTH = 0,
    SOUTH = 1,
    EAST = 2,
    WEST = 3,
}

enum Color : byte {
    RED = 1,
    GREEN = 2,
    BLUE = 3
}
```

### Bitfields

```
struct Flags {
    uint enabled : 1
    uint mode : 3
    uint level : 4
}
```

### Inheritance

```
struct Base {
    int base_field
}

struct Derived : Base {
    int derived_field
}

struct MultiDerived : Base1, Base2 {
    int own_field
}
```

### Imports

```
import "types.genny"
import "player.genny"
```

### Namespaces

```
namespace app {
    struct Player {
        int health
    }
}
```

Types in namespaces are referenced with dot notation: `app.Player`. Namespaces can be nested. Structs, enums, and classes can be nested within other structs/classes.

## Lua Scripting API

ReGenny embeds Lua 5.4 with full access to the type system and process memory.

### Key Globals

- `regenny` -- the ReGenny app instance
  - `regenny:type()` -- current selected type (sdkgenny.Type; use `:as_struct()` to get Struct methods)
  - `regenny:address()` -- current address (uintptr_t)
  - `regenny:overlay()` -- StructOverlay for current type at current address
  - `regenny:sdk()` -- the parsed SDK (Sdk object)
  - `regenny:process()` -- the attached Process

- `sdkgenny` -- type system library
  - `sdkgenny.StructOverlay(address, struct)` -- create an overlay
  - `sdkgenny.PointerOverlay(address, pointer)` -- create a pointer overlay
  - `sdkgenny.parse(code)` -- parse .genny source code string, returns Sdk
  - `sdkgenny.parse_file(path)` -- parse a .genny file, returns Sdk

### Process Memory Access

```lua
local proc = regenny:process()
local val = proc:read_uint32(0x7FF6A000)
proc:write_uint32(0x7FF6A000, 42)

-- All read/write methods:
-- proc:read_uint8/16/32/64(addr)     proc:write_uint8/16/32/64(addr, val)
-- proc:read_int8/16/32/64(addr)      proc:write_int8/16/32/64(addr, val)
-- proc:read_float(addr)              proc:write_float(addr, val)
-- proc:read_double(addr)             proc:write_double(addr, val)
-- proc:read_string(addr, do_strlen)  -- do_strlen=true for null-terminated

-- Module lookup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cursey/regenny](https://github.com/cursey/regenny) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
