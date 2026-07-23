---
trigger: always_on
description: Actionable guidelines for the iceberg-rust project, optimized for AI coding assistants.
---

# iceberg-rust: Design Principles & Patterns

Actionable guidelines for the iceberg-rust project, optimized for AI coding assistants.

## Table of Contents

1. [Project Architecture](#project-architecture)
2. [Deep vs Shallow Modules](#deep-vs-shallow-modules)
3. [LSP-Based Codebase Navigation](#lsp-based-codebase-navigation)
4. [Functional Programming Patterns](#functional-programming-patterns)
5. [Trait Design Patterns](#trait-design-patterns)
6. [Builder Pattern & Configuration](#builder-pattern--configuration)
7. [Error Handling](#error-handling)
8. [Module Organization](#module-organization)
9. [Complexity Management](#complexity-management)
10. [Quick Reference: Decision Trees](#quick-reference-decision-trees)
11. [Critical Metrics](#critical-metrics)
12. [Key Takeaways](#key-takeaways)

## Project Architecture

**Layered Design:**
```
datafusion_iceberg (query engine integration)
        ↓
  iceberg-rust (table operations, catalogs)
        ↓
iceberg-rust-spec (pure specification types)
```

**Core Philosophy:** Deep modules with simple interfaces (John Ousterhout's "A Philosophy of Software Design")

## Deep vs Shallow Modules

**Deep Modules** = Powerful functionality + Simple interface
- **Best modules** hide significant complexity behind clean APIs
- **Goal:** Minimize interface size relative to implementation size (1:10+ ratio ideal)
- **Example:** Catalog trait has ~20 methods hiding 6 implementations with 5000+ lines (1:12 ratio)

**Shallow Modules to Avoid:**
- Many small methods that just wrap other calls
- Interfaces that expose internal complexity
- Documentation longer than implementation

## LSP-Based Codebase Navigation

**IMPORTANT:** When an LSP (Language Server Protocol) MCP server is available (such as `rust-analyzer`), **ALWAYS prefer LSP tools over text-based search** for code navigation and analysis.

### When to Use LSP Tools

Use LSP tools for:
- **Finding definitions:** `get_symbol_definitions` instead of grepping for function/type names
- **Finding references:** `get_symbol_references` instead of searching for usage
- **Type information:** `get_hover` for accurate type and documentation
- **Code structure:** `get_symbols` for understanding module organization
- **Implementations:** `get_implementations` for finding trait implementations
- **Call hierarchy:** `get_call_hierarchy` for understanding call relationships
- **Diagnostics:** `get_diagnostics` for compiler errors and warnings
- **Completions:** `get_completions` for valid code suggestions

### Decision Tree

```
Need to understand code structure? → get_symbols
Need to find where something is defined? → get_symbol_definitions
Need to find all usages? → get_symbol_references
Need to understand types? → get_hover
Need to find trait impls? → get_implementations
Searching for text/patterns? → Grep/text search
```

## Functional Programming Patterns

### Guidelines

1. **Use Iterator Methods:** `map`, `filter`, `flat_map`, `fold` over `for` loops
2. **Lazy When Possible:** Return `impl Iterator` for large transformations
3. **Combinators:** `ok_or`, `and_then`, `unwrap_or_default` for `Option`/`Result`
4. **Strategic collect():** Only use `.collect::<Vec<_>>()` when needed
5. **Chain Iterators:** Use `.chain()` instead of extending vecs

### When NOT to Use Iterators

- Complex state machines (use explicit loops)
- Performance-critical hot paths needing specific optimizations
- When mutation in place is clearer

## Trait Design Patterns

### When to Create Traits

**Decision Tree:**
```
Is it used by 3+ types? → YES → Consider trait
         ↓ NO
Does it hide significant complexity? → YES → Consider trait
         ↓ NO
Would From/Into/standard trait work? → YES → Use standard trait
         ↓ NO
         → Don't create trait, use generic functions or enum
```

### Guidelines

1. **Prefer Standard Traits:** Use `From`, `TryFrom`, `Into`, `Display`, `Debug`, `Error` over custom traits
2. **Interface/Implementation Ratio:** Aim for 1:10+ lines (interface:implementation)
3. **Required Bounds:** Always include `Send + Sync + Debug` for shared types
4. **Async Traits:** Use `#[async_trait]` for I/O operations
5. **Arc Receivers:** Use `Arc<Self>` for async trait methods needing shared ownership

### Documentation Standards

Every public trait method must document:
1. **Summary:** One-line behavior description
2. **Arguments:** Each parameter explained
3. **Returns:** Success case
4. **Errors:** All failure modes

## Builder Pattern & Configuration

### When to Use Builders

**Decision Tree:**
```
5+ fields OR complex optional config OR needs async setup
         → derive_builder
Otherwise
         → Regular struct with new()
```

### Best Practices

1. **Use derive_builder:** Don't hand-roll builders
2. **Ergonomics:** Use `setter(into)` for `String` and common types
3. **Optional is Explicit:** Use `Option<T>` + `strip_option` for clarity
4. **Required Fields:** No defaults - force user to provide
5. **Validation Separate:** Use `TryInto` for validation logic
6. **Custom build():** Add async `build()` taking external dependencies

## Error Handling

### Guidelines

1. **Use thiserror:** Always derive `Error` trait, never hand-roll

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JanKaul/iceberg-rust](https://github.com/JanKaul/iceberg-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
