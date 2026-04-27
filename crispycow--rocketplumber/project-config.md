---
trigger: always_on
description: You are an expert at crafting clear, concise and idiomatic Julia code. You have an extensive understanding of this code base, and it is in your interest to maintain the highest levels of code quality within it for future maintainability.
---

# Rocket Plumber -- AI Agent Instructions

You are an expert at crafting clear, concise and idiomatic Julia code. You have an extensive understanding of this code base, and it is in your interest to maintain the highest levels of code quality within it for future maintainability.

Rocket Plumber (working title: HexFlow, still prevalent throughout the codebase) is a spaceship-building simulation game with deep fluid mechanics modelled on circuit simulation. Players construct ships from pipes and components on a hex grid, then fly them to destinations in space with realistic manoeuvre dynamics.

---

## Coding Style and Conventions

### Spelling and Language

Use **Australian English** spellings throughout (e.g., "colour", "behaviour", "manoeuvre", "initialise").

### Documentation

- **Docstrings over inline comments** for function/type documentation
  - Single-line: `"Description here"` (double quotes for single-line docstrings; keep them short)
  - Multi-line: `"""Description here"""`
- **Overly verbose docstrings**: docstrings should convey the function's purpose, not repeat its name/signature or describe step-by-step workings. Only include detailed notes when there is genuine complexity or a non-obvious quirk worth flagging
- **Base function extensions** (`Base.convert`, `Base.promote_rule`, `Base.:+`, etc.): always include docstrings explaining the behaviour -- the implementation alone can be confusing
- **Inline comments** should convey intent and explain complex logic, not narrate obvious operations

### Comment Style

The goal is to balance readability with conciseness. The appropriate level of commenting depends on code complexity:

- **Simple functions** (getters, conversions): rely on docstrings, minimal inline comments
- **Intermediate complexity** (rendering helpers, data manipulation): comment unintuitive operations and edge cases. Combine multiple related operations under a single explanatory comment
- **Complex algorithms** (graph traversal, MNA solver, physics): keep substantial explanatory comments describing the mathematical/algorithmic concepts. Reference external resources where applicable. Retain comments that help readers understand non-obvious invariants or state

**Comment placement:**
- Single line of code: inline (same line) after the code
- Few lines (2--5): comment on its own line above the block
- Larger sections: block comments `#= ... =#`

**Comment consolidation**: replace multiple single-line comments with one describing the block; move single-line explanations inline; keep "why" comments, remove "what" comments that restate obvious code.

**Struct field comments**: brief inline comments on struct fields are valuable for documentation -- keep these unless completely redundant.

### Section Dividers

Major sections:
```julia
# ===============================================================
# Section Title
# ===============================================================
```

Minor sections:
```julia
#= ---------- Section Name ----------- =#
```

### Line Length and Argument Layout

Do not excessively break function signatures across multiple lines. Follow these rules:

- **Short signatures** (fits comfortably on one line): keep everything on a single line
- **Medium signatures**: break at the `;` between positional and keyword arguments. Positional args on the first line; kwargs start on the next, each on its own indented line
- **Long signatures**: break positional args onto the first continuation line, then one kwarg per line
- **General principle**: don't split a line that reads comfortably at ~120--140 columns. Only break when a single line would genuinely be hard to scan
- The same rules apply to function calls, constructor invocations, and similar long expressions
- Exception: rendering function signatures in component definition files, where the args are always the same -- best to keep them on one line

### Code Quality

- **Readability over performance** outside hot inner loops
- **Idiomatic Julia**: use dispatch, iterators, and standard library functions (`map`, `filter`, `filter!`, `copyto!`) where they improve clarity
- **Early returns and guard clauses** to reduce nesting depth
- **Extract duplicate code** into helper functions when it improves maintainability. Do not add excessive helper functions that do not address code duplication -- they just add complexity and indirection
- **Accessor functions**: use `get_component`, `get_ship`, etc. when available instead of direct dict/struct access for better error messages and future flexibility

### Performance

- **Hot inner loops** (MNA matrix stamping per-element, particle physics per-particle): minimise allocations. Prefer in-place operations, pre-allocated buffers, `sizehint!`
- **Outer loops** (once per frame/tick): allocations are acceptable -- don't over-optimise
- **Type stability**: important in hot loops. Only suggest easy wins that don't change interfaces; note complex optimisations for future benchmarking passes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CrispyCow/RocketPlumber](https://github.com/CrispyCow/RocketPlumber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
