---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ouros is a sandboxed Python interpreter written in Rust. It parses Python code using Ruff's `ruff_python_parser` but implements its own runtime execution model for safety and performance. This is a work-in-progress project that currently supports a subset of Python features.

Project goals:

- **Safety**: Execute untrusted Python code safely without FFI or C dependencies, instead sandbox will call back to host to run foreign/external functions.
- **Performance**: Fast execution through compile-time optimizations and efficient memory layout
- **Simplicity**: Clean, understandable implementation focused on a Python subset
- **Snapshotting and iteration**: Plan is to allow code to be iteratively executed and snapshotted at each function call
- Targets the latest stable version of Python, currently Python 3.14

## Important Security Notice

It's ABSOLUTELY CRITICAL that there's no way for code run in an Ouros sandbox to access the host filesystem, or environment or to in any way "escape the sandbox".

**Ouros will be used to run untrusted, potentially malicious code.**

Make sure there's no risk of this, either in the implementation, or in the public API that makes it more like that a developer using the ouros package might make such a mistake.

Possible security risks to consider:
* filesystem access
* path traversal to access files the users did not intend to expose to the ouros sandbox
* memory errors - use of unsafe memory operations
* excessive memory usage - evading ouros's resource limits
* infinite loops - evading ouros's resource limits
* network access - sockets, HTTP requests
* subprocess/shell execution - os.system, subprocess, etc.
* import system abuse - importing modules with side effects or accessing `__import__`
* external function/callback misuse - callbacks run in host environment
* deserialization attacks - loading untrusted serialized Ouros/snapshot data
* regex/string DoS - catastrophic backtracking or operations bypassing limits
* information leakage via timing or error messages
* Python/Javascript/Rust APIs that accidentally allow developers to expose their host to ouros code

## Bytecode VM Architecture

Ouros is implemented as a bytecode VM, same as CPython.

### Reference Count Safety

All types that implement `DropWithHeap` hold heap references and **must** be cleaned up correctly on every code path — not just the happy path, but also early returns via `?`, `continue`, conditional branches, etc. A missed `drop_with_heap` on any branch leaks reference counts. There are three mechanisms for ensuring this, listed in order of preference:

#### 1. `defer_drop!` macro (preferred)

The simplest and safest approach. Use `defer_drop!` (or `defer_drop_mut!` when mutable access to the value is needed) to bind a value into a guard that automatically drops it when scope exits — whether that's normal completion, early return via `?`, `continue`, or any other branch. The macro rebinds the value and heap variables as borrows from the guard, so you keep using them by name as before:

```rust
let value = self.pop();
defer_drop!(value, heap);          // value is now &Value, heap is now &mut Heap
let result = value.py_repr(heap)?; // guard handles cleanup on all paths
```

Beyond safety, `defer_drop!` is often much more concise than inserting `drop_with_heap` calls in every branch of complex control flow.

`defer_drop!` gives you an immutable reference to the value. Use `defer_drop_mut!` when you need a mutable reference (e.g. iterators, values you may swap):

```rust
let iter = heap.get_iter(iter_ref);
defer_drop_mut!(iter, heap);
while let Some(item) = iter.for_next(heap)? { ... }
```

**Limitation:** because the macro rebinds the heap, it cannot be used inside `&mut self` methods where `self` owns the heap — first assign `let this = self;` and pass `this` instead.

#### 2. `HeapGuard` (when you need control over the value's fate)

Use `HeapGuard` directly when `defer_drop!` is too restrictive — specifically when you need to conditionally extract the value instead of dropping it. `HeapGuard` provides `into_inner()` and `into_parts()` to reclaim ownership, while its `Drop` impl still guarantees cleanup on all other paths:

```rust
// HeapGuard needed here because on success we push lhs back onto the stack
// instead of dropping it
let mut lhs_guard = HeapGuard::new(self.pop(), self);
let (lhs, this) = lhs_guard.as_parts_mut();

if lhs.py_iadd(rhs, this.heap)? {
    let (lhs, this) = lhs_guard.into_parts(); // reclaim lhs, don't drop
    this.push(lhs);
    return Ok(());
}
// otherwise lhs_guard drops lhs automatically at scope exit
```

#### 3. Manual `drop_with_heap` (for trivially simple cases)

For very simple cases with a single linear code path and no branching between acquiring and releasing the value, a direct `drop_with_heap` call is fine:

```rust
let iter = self.pop();
iter.drop_with_heap(&mut self.heap); // single path, no branching
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parcadei/ouros](https://github.com/parcadei/ouros) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
