---
trigger: always_on
description: These principles apply to **ALL** programming languages and file types.
---

# Base Development Principles

These principles apply to **ALL** programming languages and file types.

### Design Principles

1. **KISS**  (Keep It Simple, Stupid)   — Favor straightforward solutions over clever ones
2. **DRY**   (Don't Repeat Yourself)    — Extract repeated logic into reusable components
3. **YAGNI** (You Aren't Gonna Need It) — Don't build features until they're actually needed
4. **Single Responsibility**            — Each function, class, or module should do one thing well

### Guiding Mantra

> **DON'T OVERCOMPLICATE / OVERENGINEER**
>
> Write clean, readable, and maintainable code. If a solution feels complex, step back and simplify.

### Architectural Preferences

- **Prefer composition over inheritance**                — Build behavior by combining small, focused components rather than deep inheritance hierarchies
- **Use third-party packages sparingly and judiciously** — Leverage the language's standard library first; only add dependencies when they provide significant value
- **Comprehensive error handling**                       — Use appropriate error/exception mechanisms with custom error types where beneficial

### Focus Areas (Universal)

These apply regardless of language:

- Performance optimization and profiling
- Memory management awareness
- Type safety (via annotations, hints, or static typing)
- Code readability and maintainability
- Simple, effective solutions
- Async/concurrent programming patterns (where applicable)
- Always use the most modern language features and idioms
- Always apply principles like: object-oriented programming, functional programming, modular design, reactive programming.

### Output Expectations

When generating or modifying code:

- Provide clean code with appropriate type annotations/hints
- Include performance benchmarks for critical paths (when relevant)
- Offer refactoring suggestions for existing code
- Include memory/performance profiling results when relevant

---

# Code Formatting & Alignment
> **Applies to:** All files (`*.*`)

## The Universal Alignment Rule

> **When you have a vertical list of related items with a separator, align the separators into a column.**

This applies to ALL languages, ALL contexts—code, comments, docstrings, configs, everything.

### The Pattern

```
<left>   <sep> <right>
<longer> <sep> <right>
<short>  <sep> <right>
```

Where:
- **Left elements** are padded with spaces to equal length
- **Separators** (`:`, `=`, `->`, `|`, etc.) form a vertical column
- **Right elements** start at the same column

### Examples Across Contexts

```python
# Variables
name        = "Alice"
age         = 30
is_active   = True

# Docstrings / Comments
# Args:
#     file_path   : Path to the input file.
#     max_retries : Number of retry attempts.
#     timeout     : Timeout in seconds.

# Dictionaries
config = {
    "host"     : "localhost",
    "port"     : 8080,
    "debug"    : True
}
```

```typescript
// Interfaces
interface User {
    id          : number;
    name        : string;
    createdAt   : Date;
}

// Objects
const settings = {
    theme       : "dark",
    fontSize    : 14,
    autoSave    : true
};
```

```css
/* Properties */
.container {
    display         : flex;
    justify-content : center;
    align-items     : stretch;
}
```

### When to Apply

1. **Logically related items** — Group declarations, config blocks, parameter lists
2. **Vertical lists** — 2+ items that share structure
3. **Clear separators** — `:`, `=`, `->`, `|`, or similar

### When NOT to Apply

- Single items (nothing to align with)
- Unrelated declarations
- Would require excessive padding (>20 spaces)

### The Mindset

> Imagine scanning the code quickly. Can your eye jump straight to what matters? If separators zigzag, alignment helps. If it's already clear, don't force it.

## Multiline Function Signatures

When a function has **2+ parameters**, format with one parameter per line:

```
function myFunction(
    param1   : Type1,
    param2   : Type2,
    param3   : Type3  = default
) -> ReturnType {
    // body
}
```

**The pattern mirrors verbose HTML/Vue attributes:**

```html
<MyComponent
    prop1   = "value1"
    prop2   = "value2"
    :prop3  = "dynamicValue"
    @event  = "handler"
/>
```

**Why:**
- Easier to scan parameters vertically
- Clean diffs when adding/removing parameters
- Aligns with the universal separator alignment rule
- Consistent across Python, TypeScript, JavaScript, etc.

---

# Documentation Requirements
> **Applies to:** All files (`*.*`)

## Core Documentation Principles

Good documentation transcends language syntax. These principles apply to **ALL** programming languages.

---

## Function/Method Documentation

Every function, method, or callable should include:

1. **Brief description** — What the function does (one line)
2. **Parameters**        — Each parameter with its type and purpose
3. **Return value**      — What is returned and its type
4. **Exceptions/Errors** — What errors can be raised and when
5. **Example**           — Usage example for non-trivial functions

```
<doc_block_start>
Brief description of what this function does.

<params_section>
    <param_name>: <type> - Description of the parameter
    <param_name>: <type> - Description of the parameter


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robert-hoffmann/prompts](https://github.com/robert-hoffmann/prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
