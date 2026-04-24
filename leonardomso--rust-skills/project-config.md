---
trigger: always_on
description: >
---


# Rust Best Practices

Comprehensive guide for writing high-quality, idiomatic, and highly optimized Rust code. Contains 179 rules across 14 categories, prioritized by impact to guide LLMs in code generation and refactoring.

## When to Apply

Reference these guidelines when:
- Writing new Rust functions, structs, or modules
- Implementing error handling or async code
- Designing public APIs for libraries
- Reviewing code for ownership/borrowing issues
- Optimizing memory usage or reducing allocations
- Tuning performance for hot paths
- Refactoring existing Rust code

## Rule Categories by Priority

| Priority | Category | Impact | Prefix | Rules |
|----------|----------|--------|--------|-------|
| 1 | Ownership & Borrowing | CRITICAL | `own-` | 12 |
| 2 | Error Handling | CRITICAL | `err-` | 12 |
| 3 | Memory Optimization | CRITICAL | `mem-` | 15 |
| 4 | API Design | HIGH | `api-` | 15 |
| 5 | Async/Await | HIGH | `async-` | 15 |
| 6 | Compiler Optimization | HIGH | `opt-` | 12 |
| 7 | Naming Conventions | MEDIUM | `name-` | 16 |
| 8 | Type Safety | MEDIUM | `type-` | 10 |
| 9 | Testing | MEDIUM | `test-` | 13 |
| 10 | Documentation | MEDIUM | `doc-` | 11 |
| 11 | Performance Patterns | MEDIUM | `perf-` | 11 |
| 12 | Project Structure | LOW | `proj-` | 11 |
| 13 | Clippy & Linting | LOW | `lint-` | 11 |
| 14 | Anti-patterns | REFERENCE | `anti-` | 15 |

---

## Quick Reference

### 1. Ownership & Borrowing (CRITICAL)

- [`own-borrow-over-clone`](rules/own-borrow-over-clone.md) - Prefer `&T` borrowing over `.clone()`
- [`own-slice-over-vec`](rules/own-slice-over-vec.md) - Accept `&[T]` not `&Vec<T>`, `&str` not `&String`
- [`own-cow-conditional`](rules/own-cow-conditional.md) - Use `Cow<'a, T>` for conditional ownership
- [`own-arc-shared`](rules/own-arc-shared.md) - Use `Arc<T>` for thread-safe shared ownership
- [`own-rc-single-thread`](rules/own-rc-single-thread.md) - Use `Rc<T>` for single-threaded sharing
- [`own-refcell-interior`](rules/own-refcell-interior.md) - Use `RefCell<T>` for interior mutability (single-thread)
- [`own-mutex-interior`](rules/own-mutex-interior.md) - Use `Mutex<T>` for interior mutability (multi-thread)
- [`own-rwlock-readers`](rules/own-rwlock-readers.md) - Use `RwLock<T>` when reads dominate writes
- [`own-copy-small`](rules/own-copy-small.md) - Derive `Copy` for small, trivial types
- [`own-clone-explicit`](rules/own-clone-explicit.md) - Make `Clone` explicit, avoid implicit copies
- [`own-move-large`](rules/own-move-large.md) - Move large data instead of cloning
- [`own-lifetime-elision`](rules/own-lifetime-elision.md) - Rely on lifetime elision when possible

### 2. Error Handling (CRITICAL)

- [`err-thiserror-lib`](rules/err-thiserror-lib.md) - Use `thiserror` for library error types
- [`err-anyhow-app`](rules/err-anyhow-app.md) - Use `anyhow` for application error handling
- [`err-result-over-panic`](rules/err-result-over-panic.md) - Return `Result`, don't panic on expected errors
- [`err-context-chain`](rules/err-context-chain.md) - Add context with `.context()` or `.with_context()`
- [`err-no-unwrap-prod`](rules/err-no-unwrap-prod.md) - Never use `.unwrap()` in production code
- [`err-expect-bugs-only`](rules/err-expect-bugs-only.md) - Use `.expect()` only for programming errors
- [`err-question-mark`](rules/err-question-mark.md) - Use `?` operator for clean propagation
- [`err-from-impl`](rules/err-from-impl.md) - Use `#[from]` for automatic error conversion
- [`err-source-chain`](rules/err-source-chain.md) - Use `#[source]` to chain underlying errors
- [`err-lowercase-msg`](rules/err-lowercase-msg.md) - Error messages: lowercase, no trailing punctuation
- [`err-doc-errors`](rules/err-doc-errors.md) - Document errors with `# Errors` section
- [`err-custom-type`](rules/err-custom-type.md) - Create custom error types, not `Box<dyn Error>`

### 3. Memory Optimization (CRITICAL)

- [`mem-with-capacity`](rules/mem-with-capacity.md) - Use `with_capacity()` when size is known
- [`mem-smallvec`](rules/mem-smallvec.md) - Use `SmallVec` for usually-small collections
- [`mem-arrayvec`](rules/mem-arrayvec.md) - Use `ArrayVec` for bounded-size collections
- [`mem-box-large-variant`](rules/mem-box-large-variant.md) - Box large enum variants to reduce type size
- [`mem-boxed-slice`](rules/mem-boxed-slice.md) - Use `Box<[T]>` instead of `Vec<T>` when fixed
- [`mem-thinvec`](rules/mem-thinvec.md) - Use `ThinVec` for often-empty vectors
- [`mem-clone-from`](rules/mem-clone-from.md) - Use `clone_from()` to reuse allocations
- [`mem-reuse-collections`](rules/mem-reuse-collections.md) - Reuse collections with `clear()` in loops
- [`mem-avoid-format`](rules/mem-avoid-format.md) - Avoid `format!()` when string literals work
- [`mem-write-over-format`](rules/mem-write-over-format.md) - Use `write!()` instead of `format!()` 
- [`mem-arena-allocator`](rules/mem-arena-allocator.md) - Use arena allocators for batch allocations
- [`mem-zero-copy`](rules/mem-zero-copy.md) - Use zero-copy patterns with slices and `Bytes`
- [`mem-compact-string`](rules/mem-compact-string.md) - Use `CompactString` for small string optimization
- [`mem-smaller-integers`](rules/mem-smaller-integers.md) - Use smallest integer type that fits
- [`mem-assert-type-size`](rules/mem-assert-type-size.md) - Assert hot type sizes to prevent regressions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonardomso/rust-skills](https://github.com/leonardomso/rust-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
