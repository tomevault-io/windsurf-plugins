---
trigger: always_on
description: >
---


# Rust Best Practices

Comprehensive guide for writing high-quality, idiomatic, and highly optimized Rust code. Contains 265 rules across 26 categories, prioritized by impact to guide LLMs in code generation and refactoring. Current for Rust 1.96 (2024 edition).

## When to Apply

Reference these guidelines when:
- Writing new Rust functions, structs, or modules
- Implementing error handling or async code
- Writing concurrent, parallel, or `unsafe` code
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
| 3 | Memory Optimization | CRITICAL | `mem-` | 17 |
| 4 | Unsafe Code | CRITICAL | `unsafe-` | 7 |
| 5 | API Design | HIGH | `api-` | 17 |
| 6 | Async/Await | HIGH | `async-` | 18 |
| 7 | Concurrency | HIGH | `conc-` | 4 |
| 8 | Compiler Optimization | HIGH | `opt-` | 12 |
| 9 | Numeric & Arithmetic Safety | HIGH | `num-` | 5 |
| 10 | Type Safety | MEDIUM | `type-` | 13 |
| 11 | Trait & Generics Design | MEDIUM | `trait-` | 6 |
| 12 | Conversions | MEDIUM | `conv-` | 3 |
| 13 | Const & Compile-Time | MEDIUM | `const-` | 4 |
| 14 | Serde | MEDIUM | `serde-` | 8 |
| 15 | Pattern Matching | MEDIUM | `pat-` | 5 |
| 16 | Macros | MEDIUM | `macro-` | 8 |
| 17 | Closures | MEDIUM | `closure-` | 5 |
| 18 | Collections | MEDIUM | `coll-` | 4 |
| 19 | Naming Conventions | MEDIUM | `name-` | 16 |
| 20 | Testing | MEDIUM | `test-` | 15 |
| 21 | Documentation | MEDIUM | `doc-` | 12 |
| 22 | Observability | MEDIUM | `obs-` | 7 |
| 23 | Performance Patterns | MEDIUM | `perf-` | 13 |
| 24 | Project Structure | LOW | `proj-` | 14 |
| 25 | Clippy & Linting | LOW | `lint-` | 13 |
| 26 | Anti-patterns | REFERENCE | `anti-` | 15 |

---

## Quick Reference

### 1. Ownership & Borrowing (CRITICAL)

- [`own-borrow-over-clone`](rules/own-borrow-over-clone.md) - Prefer `&T` borrowing over `.clone()`
- [`own-slice-over-vec`](rules/own-slice-over-vec.md) - Accept `&[T]` not `&Vec<T>`, `&str` not `&String`
- [`own-cow-conditional`](rules/own-cow-conditional.md) - Use `Cow<'a, T>` for conditional ownership
- [`own-arc-shared`](rules/own-arc-shared.md) - Use `Arc<T>` for thread-safe shared ownership
- [`own-rc-single-thread`](rules/own-rc-single-thread.md) - Use `Rc<T>` for shared ownership in single-threaded contexts
- [`own-refcell-interior`](rules/own-refcell-interior.md) - Use `RefCell<T>` for interior mutability in single-threaded code
- [`own-mutex-interior`](rules/own-mutex-interior.md) - Use `Mutex<T>` for interior mutability across threads
- [`own-rwlock-readers`](rules/own-rwlock-readers.md) - Use `RwLock<T>` when reads significantly outnumber writes
- [`own-copy-small`](rules/own-copy-small.md) - Implement `Copy` for small, simple types
- [`own-clone-explicit`](rules/own-clone-explicit.md) - Use explicit `Clone` for types where copying has meaningful cost
- [`own-move-large`](rules/own-move-large.md) - Move large types instead of copying; use `Box` if moves are expensive
- [`own-lifetime-elision`](rules/own-lifetime-elision.md) - Rely on lifetime elision rules; add explicit lifetimes only when required

### 2. Error Handling (CRITICAL)

- [`err-thiserror-lib`](rules/err-thiserror-lib.md) - Use `thiserror` for library error types
- [`err-anyhow-app`](rules/err-anyhow-app.md) - Use `anyhow` for application error handling
- [`err-result-over-panic`](rules/err-result-over-panic.md) - Return `Result<T, E>` instead of panicking for recoverable errors
- [`err-context-chain`](rules/err-context-chain.md) - Add context with `.context()` or `.with_context()`
- [`err-no-unwrap-prod`](rules/err-no-unwrap-prod.md) - Avoid `unwrap()` in production code; use `?`, `expect()`, or handle errors
- [`err-expect-bugs-only`](rules/err-expect-bugs-only.md) - Use `expect()` only for invariants that indicate bugs, not user errors
- [`err-question-mark`](rules/err-question-mark.md) - Use `?` operator for clean propagation
- [`err-from-impl`](rules/err-from-impl.md) - Implement `From<E>` for error conversions to enable `?` operator
- [`err-source-chain`](rules/err-source-chain.md) - Preserve error chains with `#[source]` or `source()` method
- [`err-lowercase-msg`](rules/err-lowercase-msg.md) - Start error messages lowercase, no trailing punctuation
- [`err-doc-errors`](rules/err-doc-errors.md) - Document error conditions with `# Errors` section in doc comments
- [`err-custom-type`](rules/err-custom-type.md) - Define custom error types for domain-specific failures

### 3. Memory Optimization (CRITICAL)

- [`mem-with-capacity`](rules/mem-with-capacity.md) - Use `with_capacity()` when size is known
- [`mem-smallvec`](rules/mem-smallvec.md) - Use `SmallVec` for usually-small collections
- [`mem-arrayvec`](rules/mem-arrayvec.md) - Use `ArrayVec<T, N>` for fixed-capacity collections that never heap-allocate
- [`mem-box-large-variant`](rules/mem-box-large-variant.md) - Box large enum variants to reduce overall enum size

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonardomso/rust-skills](https://github.com/leonardomso/rust-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
