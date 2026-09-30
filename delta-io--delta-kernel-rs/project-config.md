---
trigger: always_on
description: The `delta_kernel_ffi` crate exposes the kernel to C/C++ via a stable FFI boundary using
---

# FFI Layer

The `delta_kernel_ffi` crate exposes the kernel to C/C++ via a stable FFI boundary using
cbindgen-generated headers (`.h` and `.hpp`).

## Handle System

Objects crossing the FFI boundary may be wrapped in **handles** -- opaque pointers with
ownership semantics:
- **Exclusive handles** (`mutable=true`, `Box`-like) -- one owner, neither `Copy` nor `Clone`
- **Shared handles** (`Arc`-like) -- shared ownership via reference counting

A handle is needed when a value might outlive the function call that passes it across the
FFI boundary, or when the type is not representable in C/C++ (dyn trait references, slices,
options, etc.). Short-lived "plain old data" types like `ExternResult`, `FFIKernelError`,
`KernelStringSlice`, and `EngineIterator` do not need handles.

Borrowed record arrays use `FfiSlice<T>`: empty slices accept null or non-null pointers, while
non-empty slices require a non-null pointer. The pointed-to storage is never owned.
Descriptive aliases identify each public array's element type.

Every handle has a corresponding `free_*` function (e.g. `free_engine`, `free_snapshot`).

Handle parameters follow one of two ownership contracts:

1. **Borrow:** Rust accesses the handle with `as_ref()` or `as_mut()`. The caller retains
   ownership and remains responsible for passing the handle to its `free_*` function.
2. **Unconditional consume:** Rust calls `into_inner()` before any fallible work. Rust owns the
   value from native entry onward and is responsible for dropping it on every result, including
   errors. The caller must not use or free the handle after the call.

Do not conditionally consume a handle only when a fallible operation succeeds. Every function's
safety documentation must state whether each handle is borrowed or consumed regardless of the
result. For consuming functions, perform string parsing, visitor decoding, validation, and other
fallible work only after all consumed handles have been converted with `into_inner()`.

## Error Handling

Fallible functions return `ExternResult` (tagged union of Ok/Err). The caller provides an
`allocate_error` callback when creating the engine; kernel calls this to allocate errors in
the caller's memory space.

## Key Files

- `src/lib.rs` -- main FFI entry points and type definitions
- `src/delta_types.rs` -- reusable borrowed C representations of Delta state and actions
- `src/handle.rs` -- opaque handle system for passing Rust objects across FFI
- `src/column_default.rs` -- column-default (`allowColumnDefaults`) reads and the write-path ack
- `src/scan.rs` -- scan FFI interface
- `src/schema_visitor.rs` -- visitor pattern for schema traversal
- `src/ffi_tracing.rs` -- log, metrics, and frame callback registration
  (`#[cfg(feature = "tracing")]`)
- `src/ffi_metrics.rs` -- `repr(C)` mirror of kernel `MetricEvent` types (`#[cfg(feature = "tracing")]`)
- `src/alloc_stats.rs` -- `peak_alloc` global allocator and native-heap FFI getters
  (`alloc-tracking`)

## Read Flow

```
get_default_engine() -> get_snapshot_builder() -> snapshot_builder_build() -> scan() -> scan_metadata() -> read + transform
```

Snapshot builder API (`ffi/src/lib.rs`):
- `get_snapshot_builder(path, engine)` -- fresh snapshot from a table path
- `get_snapshot_builder_from(old_snapshot, engine)` -- incremental update reusing an existing snapshot (avoids re-reading the log)
- `snapshot_builder_with_version(builder, version)` -- optional: pin to a specific version
- `snapshot_builder_with_log_tail(builder, log_tail)` -- optional: set log tail (for catalog-managed tables)
- `snapshot_builder_with_max_catalog_version(builder, version)` -- optional: set max catalog version (for catalog-managed tables)
- `snapshot_builder_with_snapshot_hint(builder, hint)` -- optional: validate and copy a complete
  typed snapshot hint into the builder. Log paths may name published or staged commits, checkpoint
  files, or CRC files; log compaction paths are rejected. Kernel cannot verify that supplied log
  paths belong to the builder's table, so the caller must ensure every path addresses that table.
  A failed call consumes and drops the builder
- `snapshot_builder_build(builder)` -- consume the builder and produce a `SharedSnapshot`
- `free_snapshot_builder(builder)` -- discard without building (e.g. on error paths)

Each `snapshot_builder_with_*` call consumes its input handle and returns the updated handle on
success. The caller must replace the input handle with that result. On error, the builder is
dropped. Snapshot-hint inputs and all nested pointers are borrowed only for the call and copied
into the builder. Cross-component and table validation occurs when the builder is built. The caller
must eventually pass the final returned handle to either `snapshot_builder_build` or
`free_snapshot_builder`.

Snapshot accessors (`ffi/src/lib.rs`) read a built `SharedSnapshot` without I/O -- e.g. `version`,
`snapshot_timestamp`, and `snapshot_file_stats`, which returns `OptionalValue<FfiFileStats>` (scalar
`num_files` / `table_size_bytes` from the CRC; `None` when the snapshot has no CRC, or its CRC lacks
complete file stats). `visit_file_size_histogram` exposes the optional variable-length histogram in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [delta-io/delta-kernel-rs](https://github.com/delta-io/delta-kernel-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
