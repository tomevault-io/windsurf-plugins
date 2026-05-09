---
trigger: always_on
description: Chloe is a terminal-based CLI application that mimics Auto Claude functionality, providing:
---

# Chloe - Auto Claude CLI

## Project Overview

Chloe is a terminal-based CLI application that mimics Auto Claude functionality, providing:
- **Kanban Board**: Task management with To Do, In Progress, and Done columns
- **Interactive Instances**: Multiple instance panes running actual shell sessions
- **Persistent State**: Tasks and instance configurations saved across sessions

## Safety Policy

### Zero-Tolerance for Unsafe Code

**All code in this project MUST be 100% safe Rust. No exceptions.**

This means:
- ❌ NO `unsafe` blocks
- ❌ NO `unsafe fn` definitions
- ❌ NO `unsafe impl`
- ❌ NO unsafe threading patterns
- ❌ NO raw pointer manipulation
- ❌ NO unchecked operations

### Enforcement

**Compiler-Level:**
```rust
// src/main.rs
#![forbid(unsafe_code)]
```

The `forbid` directive (stronger than `deny`) prevents ANY unsafe code, even if explicitly allowed elsewhere.

**Static Analysis:**
```toml
# Cargo.toml
[lints.rust]
unsafe_code = "forbid"

[lints.clippy]
undocumented_unsafe_blocks = "forbid"
```

### Benefits

**Memory Safety** - Rust's ownership system guarantees:
- No use-after-free
- No double-free
- No dangling pointers
- No buffer overflows
- No data races

**Thread Safety** - Safe Rust ensures:
- `Send`/`Sync` traits enforced by compiler
- No shared mutable state without synchronization
- Channels and mutexes are safe by default

**Security** - Safe Rust eliminates:
- **70% of CVEs** that affect C/C++ programs
- Entire CWE categories (CWE-119, CWE-416, CWE-476, etc.)
- Memory corruption exploits
- Race condition vulnerabilities

**Maintainability**:
- No need to audit unsafe blocks
- Refactoring is safe by default
- New contributors can't introduce memory bugs
- Compiler is your safety net

### Dependencies Using Unsafe

Some dependencies (like `portable-pty`) internally use `unsafe` to interact with OS APIs. This is acceptable when:

1. **Public API is safe**: We only call safe functions
2. **Well-maintained**: Dependency is actively maintained
3. **Necessary**: No safe alternative exists for OS interaction
4. **Encapsulated**: Unsafe code is isolated and reviewed by experts

We **never** call:
- `unsafe` functions from dependencies
- FFI functions directly
- C libraries without safe wrappers

### Verification

```bash
# Should compile (no unsafe code)
cargo build

# Should pass (static analysis)
cargo clippy

# Grep for unsafe (should find nothing in src/)
grep -r "unsafe" src/
```

### There Are NO Exceptions

If you believe you need unsafe code:
1. You're probably wrong - find a safe alternative
2. If truly necessary, move it to a separate crate with extensive documentation
3. That crate will NOT be part of this repository

**Remember: If it doesn't compile with `#![forbid(unsafe_code)]`, it doesn't ship.**

## Code Quality Standards

### Comments: Only "Why", Never "How" or "What"

**Golden Rule**: Code should be self-documenting. Comments explain *why*, never *what* or *how*.

```rust
// ❌ BAD: "What" comment - code already says this
// Increment the counter
counter += 1;

// ❌ BAD: "How" comment - code already shows how
// Loop through items and find matching ID
for item in items {
    if item.id == target_id {
        return Some(item);
    }
}

// ✅ GOOD: "Why" comment - explains business logic
// Task must move to previous column before deletion to maintain audit trail
move_task_previous();
delete_task();
```

**If you need a "how" or "what" comment, REFACTOR instead:**

```rust
// ❌ BAD: Needs comment to explain
// Parse the timestamp and convert to local timezone
let dt = chrono::DateTime::parse_from_rfc3339(&s)
    .map(|d| d.with_timezone(&chrono::Local))?;

// ✅ GOOD: Function name makes it clear
let dt = parse_timestamp_as_local(&s)?;
```

**When "why" comments are appropriate:**
- Business logic rationale: "We use base64 to avoid URL encoding issues"
- Performance trade-offs: "Caching here reduces API calls by 90%"
- Non-obvious algorithms: "Binary search chosen for O(log n) lookup on sorted data"
- Workarounds: "Clippy false positive on this pattern, see issue #123"
- Safety invariants: "SAFETY: This assumes the buffer is always initialized"

### No Abbreviations

**All identifiers must use full words, not abbreviations.**

```rust
// ❌ BAD: Abbreviations
let cfg = Config::default();
let msg = "Hello";
let idx = 0;
let btn = Button::new();
let ctx = AppContext::new();

// ✅ GOOD: Full words
let config = Config::default();
let message = "Hello";
let index = 0;
let button = Button::new();
let context = AppContext::new();
```

**Exceptions (industry standard abbreviations only):**
- `id` (identifier) - universally understood
- `url` (Uniform Resource Locator) - more common than "address"
- `html`, `json`, `xml` - file format names
- `io` (input/output) - standard library convention
- `uuid` (Universally Unique Identifier) - standard acronym
- `pty` (pseudo-terminal) - standard Unix term
- `terminal` (when referring to terminal emulator/UI) - standard term

**Common violations to avoid:**
- ❌ `num` → ✅ `number` or `count`
- ❌ `str` → ✅ `string` (except `&str` type)
- ❌ `arr` → ✅ `array`
- ❌ `btn` → ✅ `button`
- ❌ `msg` → ✅ `message`
- ❌ `tmp` → ✅ `temporary`
- ❌ `val` → ✅ `value`
- ❌ `cfg` → ✅ `config`
- ❌ `ctx` → ✅ `context`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KevinEdry/chloe](https://github.com/KevinEdry/chloe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
