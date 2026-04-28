---
trigger: always_on
description: This repository uses Zig 0.15.2 and Ghostty's `ghostty-vt` library.
---

# ghostty-opentui Agent Guide

This repository uses Zig 0.15.2 and Ghostty's `ghostty-vt` library.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│  TypeScript (tui/ffi.ts)                                                │
│  - Loads .node file via require()                                       │
│  - Calls native functions: ptyToJson, ptyToText, ptyToHtml              │
│  - Converts JSON output to typed TerminalData                           │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  N-API Bridge (napigen)                                                 │
│  - Converts JS strings to Zig slices                                    │
│  - Converts Zig return values to JS                                     │
│  - Handles errors as JS exceptions                                      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  Zig Library (src/lib.zig)                                              │
│  - Creates Ghostty Terminal instance                                    │
│  - Feeds ANSI input through VT stream                                   │
│  - Extracts styled spans from terminal buffer                           │
│  - Outputs JSON with colors, styles, cursor position                    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  Ghostty VT Emulator (deps/ghostty)                                     │
│  - Full terminal emulation (not just ANSI parsing)                      │
│  - Handles cursor movement, scrolling, line wrapping                    │
│  - Maintains screen buffer with cells, styles, colors                   │
│  - Supports 16/256/RGB colors, bold, italic, underline, etc.            │
└─────────────────────────────────────────────────────────────────────────┘
```

## napigen

[napigen](https://github.com/cztomsik/napigen) is used to create N-API bindings for the Zig code. This allows the Zig library to be loaded as a native Node.js addon (.node file).

### How it works

1. **Define functions in Zig** - Regular Zig functions that accept/return supported types
2. **Register with napigen** - Use `comptime { napigen.defineModule(initModule); }`
3. **Export functions** - In `initModule`, use `js.createFunction()` and `js.setNamedProperty()`
4. **Build outputs .node file** - `zig build` produces `ghostty-opentui.node`
5. **Load in JS** - Use `require()` to load and call functions directly

### Supported types

- Primitives: `i32`, `u32`, `bool`, etc. (maps to JS numbers/booleans)
- Strings: `[]const u8` (maps to JS strings)
- Return strings by allocating and returning `[]const u8`

### Example

```zig
// src/lib.zig
const napigen = @import("napigen");

fn add(a: i32, b: i32) i32 {
    return a + b;
}

fn greet(name: []const u8) ![]const u8 {
    // Return allocated string (caller must handle memory)
    return std.fmt.allocPrint(allocator, "Hello, {s}!", .{name});
}

comptime {
    napigen.defineModule(initModule);
}

fn initModule(js: *napigen.JsContext, exports: napigen.napi_value) !napigen.napi_value {
    try js.setNamedProperty(exports, "add", try js.createFunction(add));
    try js.setNamedProperty(exports, "greet", try js.createFunction(greet));
    return exports;
}
```

```typescript
// Usage in TypeScript (Bun)
// IMPORTANT: Do NOT use createRequire - it breaks bun compile
// Bun provides global require() in ESM modules
const native = require("./zig-out/lib/ghostty-opentui.node")

native.add(1, 2)        // => 3
native.greet("World")   // => "Hello, World!"
```

### Testing with napigen

napigen requires N-API symbols that only exist when loaded as a Node addon. For Zig tests, we conditionally exclude napigen:

```zig
const builtin = @import("builtin");
const napigen = if (builtin.is_test) undefined else @import("napigen");

// Only define module when not testing
comptime {
    if (!builtin.is_test) {
        napigen.defineModule(initModule);
    }
}
```

The `build.zig` creates a separate test module without napigen imports.

## Ghostty VT vs node-pty

These are **different things** that complement each other:

| node-pty | Ghostty VT Emulator |
|----------|---------------------|
| Spawns processes (bash, etc.) | Parses ANSI escape sequences |
| Creates pseudo-terminal (PTY) | Maintains screen buffer |
| Raw bytes in/out | Tracks cursor, colors, styles |
| No parsing or rendering | Handles scrolling, wrapping |

For interactive terminals, you'd use both:
- node-pty spawns the shell and gives raw output
- Ghostty VT parses that output into renderable state

Currently this library is **read-only** - it parses ANSI but doesn't maintain persistent state between calls. For interactive use, we'd need to add persistent terminal instances (see README for future plans).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remorses/ghostty-opentui](https://github.com/remorses/ghostty-opentui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
