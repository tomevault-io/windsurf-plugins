---
trigger: always_on
description: **Golden Rule**: Code should be self-documenting. Comments explain *why*, never *what* or *how*.
---

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
- ❌ `doc` → ✅ `document`
- ❌ `img` → ✅ `image`

**Why this matters:**
- Code is read 10x more than written
- Abbreviations are ambiguous (`msg` = message or messages?)
- IDEs have autocomplete - no typing savings
- Newcomers understand full words instantly
- Consistency > brevity

**Refactoring over Comments:**
If you're writing a comment to explain what code does, the code is too complex. Refactor by:
1. Extract to well-named functions
2. Use descriptive variable names
3. Simplify complex expressions
4. Break long functions into smaller pieces

### Never Nester: Return Early

**CRITICAL RULE: Maximum nesting depth is 2 levels. NO EXCEPTIONS.**

Nested ifs are unreadable. Use early returns and descriptive variable names:

```rust
// ❌ BAD: Nested ifs to oblivion - WHO CAN READ THIS?
if self.selected_column == 1 {
    if let Some(task_idx) = self.selected_task {
        if let Some(task) = self.columns[self.selected_column].tasks.get_mut(task_idx) {
            if let Some(instance_id) = task.instance_id {
                task.is_paused = !task.is_paused;
                return Some((instance_id, task.is_paused));
            }
        }
    }
}
None

// ✅ GOOD: Early returns with descriptive checks
let is_in_progress_column = self.selected_column == 1;
if !is_in_progress_column {
    return None;
}

let task_index = self.selected_task?;
let task = self.columns[self.selected_column].tasks.get_mut(task_index)?;
let instance_id = task.instance_id?;

task.is_paused = !task.is_paused;
Some((instance_id, task.is_paused))
```

**Key techniques:**
1. **Use descriptive variable names for conditions** - `is_in_progress_column` instead of `self.selected_column == 1`
2. **Early return/guard clauses** - Check failure conditions first and return
3. **Use `?` operator** - Convert `Option`/`Result` to early returns automatically
4. **Invert conditions** - `if !condition { return }` keeps happy path unindented

```rust
// ❌ BAD: Triple nested
if user.is_admin {
    if project.is_active {
        if has_permission {
            do_work();
        }
    }
}

// ✅ GOOD: Flat with descriptive checks
let is_admin = user.is_admin;
let is_active = project.is_active;

if !is_admin || !is_active || !has_permission {
    return;
}

do_work();
```

### Functional Core, Imperative Shell

Separate pure logic from side effects:

```rust
// FUNCTIONAL CORE: Pure business logic
fn calculate_discount(user: &User, amount: f64) -> f64 {
    // Pure function - no I/O, no mutation
    if user.is_premium && amount > 100.0 {
        amount * 0.2
    } else {
        0.0
    }
}

// IMPERATIVE SHELL: Side effects at edges
fn apply_discount_and_save(user_id: Uuid, amount: f64) -> Result<()> {
    let user = database.load(user_id)?;          // I/O

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KevinEdry/nestjs-trpc](https://github.com/KevinEdry/nestjs-trpc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
