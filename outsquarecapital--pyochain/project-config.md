---
trigger: always_on
description: The following documentation provides an overview of the project architecture and code conventions.
---

# Pyochain architecture and conventions

The following documentation provides an overview of the project architecture and code conventions.

It's destined to both humans and LLMs, and the fact that it's in `AGENTS.md` is merely a convenience to enforce it in the local context when working with agents.

## Architecture

The root crate assembles the public extension module.

`src/lib.rs` is the single registration point for the python package, and also for functionality that need to be run at import time.

### Internal Crates

We use several internal crates for faster compile times and better modularity.

If you want to find/edit/add some code, match your task to it. Does the code:

- extend `pyo3` with traits extensions, new standard-library helpers/types, ffi, etc? => [`pyo3_ext`](crates\pyo3_ext)
- is a proc macro? => [`pyochain_macros`](crates\pyochain_macros)
- extends rust std? => [`std_tools`](crates\std_tools)
- is directly related to sorted collections internal implementations? => [`sorted_rs`](crates\sorted_rs)
- is related to pyochain's public API or high-level behavior? => [`src`](src)

## Code Guidelines

### Style

#### General

The guidelines below apply to both rust source code and python scripts.

1. We use a descending order for organizing code: types, then public symbols, then private helpers.
2. Prefer chained method calls over intermediate variables. As such, generous use of `tap::{pipe, tap}` crate across the code is encouraged.
3. It should come with no surprise, given the library purpose, that we don't like imperative style `for i in iterable if condition` and vastly prefer `iterable.into_iter().filter(...).map(...)`.
4. We generally avoid inline comments (`//`, `#`). They should only be used for local implementation details, or `TODO:`/`NOTE:`/`BUG` annotations.
5. Documentation comments (`///`, `"""..."""`) are mandatory in python stubs, but outside of that should be used judiciously. They should focus on intended usage, behavior and eventual tips, rather than implementation details.
6. Functionality locality matters: free functions should be avoided if it can clearly be linked to an appropriate type (as associated function/method, classmethod/staticmethod, etc...).
7. Prefer `if condition {...} else {...}` over early returns.

#### Rust-specifics

- macros are powerful, macros are cool, but LSP and formatters don't understand them well. If you can, extract as much as possible out of it. For example, `fn foo{...}` can become `fn foo{helper_function(...)}`. Apply this with good sense however, and avoid over-engineering trivial cases.
- The function locality with traits has a special case: since a pub trait imply all methods are public, free functions can be used more liberally to keep public interfaces clean and concise.

### Implementations

#### local abstractions

Use the existing abstractions before adding a local equivalent.

- `py_abc` is one of the main abstractions for defining Python-visible behavior in Rust. Consult related documentation for proper usage.
- Raw accesses like `obj.callmethod(...)` are almost never needed. If you find yourself using it, consider implementing it in `pyo3_ext` instead.

#### Pyo3

- Prefer `self, py: Python<...>` as the default signature for methods that need Python context. `slf: Bound<'_, Self>` can be used when necessary.
- A pyclass should be frozen by default. If not, it must be explictly motivated by a clear reason.

#### python -> rust conversions

There's a LOT of different ways to convert a ``Bound<`_, PyAny>`` to a Rust type. match the situation to choose the most appropriate method.

##### Multiple conversions

| Converter                         | to                                                        | Pros                                                                                                      | Cons                                                                                                                   | Use when                                                                                                                                           | Tips                                                                                                                                                         |
| --------------------------------- | --------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OutSquareCapital/pyochain](https://github.com/OutSquareCapital/pyochain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
