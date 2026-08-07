---
trigger: always_on
description: When making code changes, always prefer correctness over shortcuts. Do not
---

# AGENTS.md

## Engineering Priorities

When making code changes, always prefer correctness over shortcuts. Do not
paper over bugs, weaken tests, hard-code narrow cases, or choose expedient
workarounds when a sound fix is within scope; take the time to preserve the
intended semantics and verify the change appropriately.

Prefer the implementation path that leads to the highest quality code over the
shortest path to a working change. When a quick fix would make later design,
ownership, or semantic boundaries worse, choose the more durable approach and
keep the scope focused on the requested outcome.

## Zig Development

Use `zigdoc` to discover current APIs for the Zig standard library and any third-party dependencies before coding.

Examples:
```bash
zigdoc std.fs
zigdoc std.posix.getuid
zigdoc vaxis.Window
```

## Validation

Use these commands for repository validation:

```bash
zig fmt .
zig build compile-check
zig build lint
zig build test
zig build conformance
```

## POSIX Conformance Target

Rush `--posix` targets POSIX.1-2024 / Issue 8 unless a test or code comment
explicitly documents a deliberate compatibility choice. When checking shell
language requirements, prefer the Issue 8 text at `pubs.opengroup.org` over
older Issue 7 / POSIX.1-2017 references.

When running conformance comparisons against other shells, use only the primary
installed-base reference shells below unless the user explicitly asks for a
platform-specific shell:

```bash
zig build conformance -- --shell dash --mode posix
zig build conformance -- --shell bash --shell-arg --posix --mode posix
zig build conformance -- --shell yash --mode posix
```

The full conformance suite intentionally prints compact one-line failures. Add
`--diff` for detailed stdout/stderr diffs, or add `--case TEXT` to run matching
case names only; `--case` implies detailed diffs.

Conformance comparisons are primarily a Rush validation tool, not a requirement
that every reference shell pass every case. If Rush matches the documented POSIX
target or an intentional compatibility choice, a reference-shell mismatch is
acceptable evidence about that shell's behavior. Do not weaken a test solely to
make dash, Bash, or yash pass; only relax expectations when the relaxed output is
still the intended Rush contract.

The Bash reference is current stable GNU Bash in POSIX mode; do not substitute
macOS `/bin/bash` 3.2 or `/bin/sh` for it. The yash reference is current yash in
POSIX mode. If one of these shells is not installed, report that it was not run
rather than silently using another shell. FreeBSD `/bin/sh`, macOS `/bin/sh`,
BusyBox ash, ksh, and other shells are secondary/platform references only.

## Current Zig Patterns

**ArrayList:**
```zig
var list: std.ArrayList(u32) = .empty;
defer list.deinit(allocator);
try list.append(allocator, 42);
```

**HashMap/StringHashMap (default to unmanaged):**
```zig
var map: std.StringHashMapUnmanaged(u32) = .empty;
defer map.deinit(allocator);
try map.put(allocator, "key", 42);
```

**stdout/stderr writer:**
```zig
var buf: [4096]u8 = undefined;
var writer = std.fs.File.stdout().writer(&buf);
defer writer.interface.flush() catch {};
try writer.interface.print("hello {s}\n", .{"world"});
```

**build.zig executable:**
```zig
b.addExecutable(.{
    .name = "foo",
    .root_module = b.createModule(.{
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    }),
});
```

**JSON writing:**
```zig
var buf: [4096]u8 = undefined;
var writer = std.fs.File.stdout().writer(&buf);
defer writer.interface.flush() catch {};

var jw: std.json.Stringify = .{
    .writer = &writer.interface,
    .options = .{ .whitespace = .indent_2 },
};
try jw.write(my_struct);
```

**Allocating writer:**
```zig
var writer: std.Io.Writer.Allocating = .init(allocator);
defer writer.deinit();
try writer.writer.print("hello {s}", .{"world"});
const output = try writer.toOwnedSlice();
```

## Zig Style

- `camelCase` for functions and methods
- lower-case `snake_case` for variables, parameters, and constants
- `PascalCase` for types, structs, and enums
- prefer `const foo: Type = .{ .field = value };` over `const foo = Type{ .field = value };`
- pass allocators explicitly; use `errdefer` for cleanup on error
- keep tests inline with the code they cover; register them in `src/main.zig`
- keep lines under 120 characters
- prefer Zig multiline string literals over string concatenation with `++` for
  long strings

### Files and Types

- Treat every `.zig` file as a namespace. Make the file itself a type only when
  its root represents one primary stateful abstraction with fields and methods.
- Name a file-backed type `PascalCase.zig`, import it directly with
  `const Widget = @import("Widget.zig");`, and begin it with an optional `//!`
  container doc followed by `const Widget = @This();`. Prefer the concrete type
  name over `Self` so signatures remain clear out of context.
- Use lower-case `snake_case.zig` for namespace modules: related free functions,
  constants, multiple peer types, or package facades. Export named types from
  these modules and import them as `@import("widget.zig").Widget`.
- Do not put a sole `pub const Widget = struct { ... };` inside `Widget.zig`;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rockorager/rush](https://github.com/rockorager/rush) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
