---
trigger: always_on
description: This document provides instructions for LLM agents contributing to libcudf-rs,
---

# Contributing to libcudf-rs: Guide for LLM Agents

This document provides instructions for LLM agents contributing to libcudf-rs,
a Rust wrapper for NVIDIA's cuDF GPU-accelerated DataFrame library.

## Architecture

This project is divided in the following crates:

- `libudf-sys`: thin Rust wrapper on top of https://github.com/rapidsai/cudf. It uses `cxx` for binding to the C++
  code of the original project, and aims to be an almost 1:1 match with the original project. Keep this crate
  simple and very thin, mostly just plumbing. Any actual public API design should happen in the root project's `src/`.
- `libcudf-rs`: root project that adds an API layer on top of `libcudf-sys` for better UX. This root crate should
  closely mimic other bindings in the original https://github.com/rapidsai/cudf project, like the Java or Python ones.

## Contributing code

- The code should be simple and comprehensive. The fewer the code, the better. If at some point you produce a big
  quantity of code, iterate over it and try to reduce it and make it simpler.
- Do not add irrelevant comments that just explain what can already be seen in the code. Just add a comment for
  pieces of code that are not immediately obvious.
- Do not try to reinvent the wheel. This project is not supposed to be creative or disruptive, it's just a plumbing
  layer on top of https://github.com/rapidsai/cudf so that it's accessible from Rust.
- Any public entity should be documented, but be brief and concise while documenting it. Quality is way preferable
  than quantity.
- The cuDF source code is downloaded during the build process and placed in the build output directory (`target/`). You
  can
  find the exact location with: `find target/ -type d -name "cudf-*" | grep -E "cudf-[0-9]+\.[0-9]+\.[0-9]+$" | head -1`

### libcudf-sys: The Thin Wrapper Layer

**CRITICAL: This crate must be a 1:1 mapping to cuDF C++ API with ZERO opinions.**

Rules for `libcudf-sys`:

- **NO type conversions** - If cuDF returns a specific type, expose that exact type
- **NO switch statements** - Don't handle different data types, let the caller do it
- **NO API design** - Just expose what cuDF provides, nothing more
- **NO helper functions** - If cuDF has `cudf::make_max_aggregation()`, expose it as-is
- **NO value judgments** - Don't decide what's "better" for the user
- **Methods vs Functions** - Match cuDF exactly:
    - If cuDF has a class method (e.g., `table_view::column()`), expose as method (e.g., `Table::get_column()`)
    - If cuDF has a free function (e.g., `cudf::reduce()`), expose as free function (e.g., `reduce()`)
    - Constructors become factory functions (e.g., `groupby::groupby()` → `groupby_create()`)

Mandatory parity gate for every `libcudf-sys` change:

1. Start from the exact local cuDF/RMM headers used by this build. Locate them with
   `find target/ -type d -name "cudf-*" | grep -E "cudf-[0-9]+\.[0-9]+\.[0-9]+$" | head -1`.
   Do not rely on memory, Python docs, Java docs, or older RAPIDS versions.
2. Identify the upstream C++ declaration before writing Rust or bridge code. The sys API must preserve the upstream
   shape: function vs method, overloads, parameter order, required parameters, defaulted parameters, enum values,
   return type, ownership, metadata, null handling, stream arguments, memory-resource arguments, and error behavior.
3. Do not drop upstream parameters in sys. If cuDF accepts an option, policy, stream, memory resource, output type,
   initial value, metadata output, or predicate handle, sys must expose it. Default-only convenience belongs in
   `libcudf-rs`, not in `libcudf-sys`.
4. Do not collapse upstream return shapes. If cuDF returns a pair, vector, table-with-metadata, device vector, scalar,
   or aggregation result, sys must preserve that shape as directly as `cxx` allows.
5. If `cxx` cannot expose an upstream C++ type directly, add the smallest opaque wrapper or bridge container needed.
   The wrapper must map directly to the upstream type's fields, ownership, and lifetime. It must not add filtering,
   conversion, defaulting, aggregation, indexing policy, or other convenience behavior.
6. RMM/CUDA support types in sys must also be upstream-shaped wrappers: streams, stream views, resource refs, and memory
   resources. Do not invent lifecycle, pooling, synchronization, or allocation behavior unless it directly wraps an
   upstream type or method.
7. Every enum exposed by sys must be numerically aligned with the upstream enum. Add parity checks for enum surfaces
   that are easy to drift.
8. Tests should lock binding parity where drift is likely: enum discriminants, overload presence, omitted-parameter
   risks, output shape/metadata preservation, and ownership/handle plumbing. Do not add tests that only restate a local
   helper implementation.
9. If a perfect upstream-shaped binding is impossible, document the `cxx` limitation in code and keep the deviation
   minimal and mechanical. Any ergonomic API must be added in `libcudf-rs`.

Example of what NOT to do:

```cpp
// ❌ BAD: Adding type conversion logic
double column_max(const Column& column) {
    auto result = cudf::reduce(...);
    switch (result->type().id()) {  // NO! This is API design

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabotechs/libcudf-rs](https://github.com/gabotechs/libcudf-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
