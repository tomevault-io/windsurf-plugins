---
trigger: always_on
description: _solve problems correctly the first time_. When you encounter a potential latency spike or algorithmic issue, fix it now—don't defer. The second pass may never come.
---

Engineering Notes

### 1. **Zero Technical Debt Policy**

_solve problems correctly the first time_. When you encounter a potential latency spike or algorithmic issue, fix it now—don't defer. The second pass may never come.

### 2. **Static Memory Allocation**

This is huge for a UI framework:

- **No dynamic allocation after initialization**
- Pre-allocate pools for glyphs, render commands, widgets at startup
- Use fixed-capacity arrays/pools instead of growing `ArrayList`s during rendering

For Gooey, this means: glyph caches, command buffers, and clip stacks should have fixed upper bounds allocated at init time. This eliminates allocation jitter during frame rendering.

### 3. **Assertion Density**

**minimum 2 assertions per function**. For Gooey:

- Assert glyph bounds before atlas insertion
- Assert clip rect validity before pushing to stack
- **Pair assertions**: assert data validity when writing to GPU buffer AND when reading back
- Assert compile-time constants (e.g., `comptime { assert(@sizeOf(Vertex) == 32); }`)
- **Split compound assertions**: prefer `assert(a); assert(b);` over `assert(a and b);` — the former is simpler to read, and provides more precise information if the condition fails
- Use single-line `if` to assert an implication: `if (a) assert(b);`
- On occasion, use a blatantly true assertion instead of a comment as stronger documentation where the assertion condition is critical and surprising

### 4. **Put a Limit on Everything**

Every loop, every queue, every buffer needs a hard cap:

```example.zig
const MAX_GLYPHS_PER_FRAME = 65536;
const MAX_CLIP_STACK_DEPTH = 32;
const MAX_NESTED_COMPONENTS = 64;
```

This prevents infinite loops and tail latency spikes. If you hit a limit, **fail fast**.

### 5. **70-Line Function Limit**

Hard limit. Split large render functions by:

- Keeping control flow (switches, ifs) in parent functions
- Moving pure computation to helpers
- "Push ifs up, fors down"
- **Good function shape** is the inverse of an hourglass: a few parameters, a simple return type, and a lot of meaty logic between the braces
- **Centralize state manipulation**: let the parent function keep all relevant state in local variables, and use helpers to compute what needs to change, rather than applying the change directly. Keep leaf functions pure

### 6. **Explicit Control Flow**

- No recursion (important for component trees—use explicit stacks)
- Minimize abstractions ("abstractions are never zero cost")
- Avoid `async`/suspend patterns that hide control flow
- **Split compound conditions**: split complex `else if` chains into nested `else { if {} }` trees to make branches and cases explicit. Consider whether a single `if` also needs a matching `else` branch to ensure positive and negative spaces are handled or asserted
- **Functions must run to completion** without suspending, so that precondition assertions remain true throughout the lifetime of the function

### 7. **Back-of-Envelope Performance Sketches**

Before implementing, sketch resource usage:

- How many vertices per frame? (GPU bandwidth)
- How many texture uploads per frame? (memory bandwidth)
- How many glyph cache lookups? (CPU/cache locality)

Optimize for **network → disk → memory → CPU** (slowest first), adjusted for frequency. A memory cache miss may be as expensive as a disk fsync if it happens many times more often.

### 8. **Batching as Religion**

We're already doing this with GPU commands, but:

- Don't react to events directly—batch them
- Amortize costs across frames
- Let the CPU sprint on large chunks, not zig-zag on tiny tasks
- **Distinguish control plane from data plane**: a clear delineation through batching enables a high level of assertion safety without losing performance. Assert heavily on the control plane; batch tightly on the data plane

### 9. **Naming Discipline**

- Units/qualifiers last: `offset_pixels_x`, `latency_ms_max`
- Same-length related names for visual alignment: `source`/`target` not `src`/`dest`
- Callbacks go last in parameter lists
- **Do not abbreviate** variable names (unless the variable is a primitive integer in a sort function or matrix calculation)
- **Infuse names with meaning**: `gpa: Allocator` and `arena: Allocator` are better than `allocator: Allocator` — they inform the reader whether `deinit` should be called
- **Don't overload names** with multiple meanings that are context-dependent
- **Think of how names read outside the code** — in docs or conversation. Nouns compose better than adjectives or participles: `pipeline` over `preparing`
- **Named arguments via `options: struct`** pattern when arguments can be mixed up. A function taking two `u64`s must use an options struct
- **Struct field ordering**: fields first, then types, then methods. Important things near the top. `main` goes first in a file
- **Descriptive commit messages** that inform and delight the reader. PR descriptions are not stored in git history and are invisible in `git blame` — they are not a replacement for commit messages

### 10. **Shrink Scope Aggressively**

- Declare variables at smallest possible scope
- Calculate/check values close to use (avoid POCPOU bugs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duanebester/gooey](https://github.com/duanebester/gooey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
