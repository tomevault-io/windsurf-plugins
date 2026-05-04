---
trigger: always_on
description: **⚠️ WARNING: Mission-critical financial infrastructure - bugs cause fund loss.**
---

# CLAUDE.md

## MISSION CRITICAL SOFTWARE

**⚠️ WARNING: Mission-critical financial infrastructure - bugs cause fund loss.**

Every line of code must be correct. Zero error tolerance.

## Core Protocols

### Working Directory

**ALWAYS run commands from the repository root directory.** Never use `cd` except when debugging a submodule. All commands, builds, and tests are designed to run from root.

### Security

- Sensitive data detected (API keys/passwords/tokens): abort, explain, request sanitized prompt
- Memory safety: plan ownership/deallocation for every allocation
- Every change must be tested and verified
- Use SafetyCounter for infinite loop prevention (300M instruction limit)
- **CRITICAL: Crashes are SEVERE SECURITY BUGS** - Any crash (e.g., from `std.debug.assert`) indicates memory unsafety or missing validation. The EVM must ALWAYS return errors gracefully, never crash. Before fixing the bug that triggered the crash, FIRST fix the validation/error handling that allowed the crash to occur.

### Build Verification

**EVERY code change**: `zig build && zig build test-opcodes`
**Exception**: .md files only

Follow TDD

### Debugging

- Bug not obvious = improve visibility first
- Use differential tests with revm in test/differential

### Zero Tolerance

❌ Broken builds/tests
❌ Stub implementations (`error.NotImplemented`)
❌ Commented code (use Git)
❌ Test failures
❌ Invalid benchmarks
❌ `std.debug.print` in modules (use `log.zig`)
❌ `std.debug.assert` (use `tracer.assert()`)
❌ Skipping/commenting tests
❌ Any stub/fallback implementations
❌ **Swallowing errors with `catch` (e.g., `catch {}`, `catch &.{}`, `catch null`)**

**STOP and ask for help rather than stubbing.**

**WHY PLACEHOLDERS ARE BANNED**: Placeholder implementations create ambiguity - the human cannot tell if "Coming soon!" or simplified output means:
1. The AI couldn't solve it and gave up
2. The AI is planning to implement it later
3. The feature genuinely isn't ready yet
4. There's a technical blocker

This uncertainty wastes debugging time and erodes trust. Either implement it fully, explain why it can't be done, or ask for help. Never leave placeholders that pretend to work.

**NEVER swallow errors! Every error must be explicitly handled or propagated. Using `catch` to ignore errors can cause silent failures and fund loss.**

## Coding Standards

### Principles

- Minimal else statements
- Single word variables (`n` not `number`)
- Direct imports (`address.Address` not aliases)
- Tests in source files
- Defer patterns for cleanup
- Always follow allocations with defer/errDefer
- Descriptive variables (`top`, `value1`, `operand` not `a`, `b`)
- Logging: use `log.zig` (`log.debug`, `log.warn`)
- Assertions: `tracer.assert(condition, "message")`
- Stack semantics: LIFO order (first pop = top)

### Memory Management

```zig
// Pattern 1: Same scope
const thing = try allocator.create(Thing);
defer allocator.destroy(thing);

// Pattern 2: Ownership transfer
const thing = try allocator.create(Thing);
errdefer allocator.destroy(thing);
thing.* = try Thing.init(allocator);
return thing;
```

### ArrayList API (Zig 0.15.1)

**CRITICAL**: In Zig 0.15.1, `std.ArrayList(T)` returns an UNMANAGED type that requires allocator for all operations!

```zig
// CORRECT: std.ArrayList is UNMANAGED (no internal allocator)
var list = std.ArrayList(T){};  // Default initialization
// OR
const list = std.ArrayList(T).empty;  // Empty constant
// OR with capacity
var list = try std.ArrayList(T).initCapacity(allocator, 100);

// All operations REQUIRE allocator:
defer list.deinit(allocator);  // ✅ allocator REQUIRED
try list.append(allocator, item);  // ✅ allocator REQUIRED
try list.ensureCapacity(allocator, 100);  // ✅ allocator REQUIRED
_ = list.pop();  // No allocator needed for pop

// Direct access (no allocator needed):
list.items[0] = value;
list.items.len = 0;

// WRONG - This does NOT work in Zig 0.15.1:
var list = std.ArrayList(T).init(allocator);  // ❌ No init() method!
list.deinit();  // ❌ Missing required allocator
try list.append(item);  // ❌ Missing required allocator

// For managed ArrayList with internal allocator, use array_list module directly:
const array_list = @import("std").array_list;
var list = array_list.AlignedManaged(T, null).init(allocator);
defer list.deinit();  // No allocator needed for managed version
```

## Testing Philosophy

- NO abstractions - copy/paste setup
- NO helpers - self-contained tests
- Test failures = fix immediately
- Evidence-based debugging only
- **CRITICAL**: Zig tests output NOTHING when passing (no output = success)
- If tests produce no output, they PASSED successfully
- Only failed tests produce output

### Debug Logging in Tests

Enable with:
```zig
test {
    std.testing.log_level = .debug;
}
```

**IMPORTANT**: Even with `std.testing.log_level = .debug`, if the test passes, you will see NO OUTPUT. This is normal Zig behavior. No output means the test passed.

## Project Architecture

### Guillotine EVM

High-performance EVM: correctness, minimal allocations, strong typing.

### Module System

Use `zig build test` not `zig test`. Common error: "primitives" package requires module system.

### Key Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evmts/guillotine](https://github.com/evmts/guillotine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
