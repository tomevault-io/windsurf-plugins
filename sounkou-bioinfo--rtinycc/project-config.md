---
trigger: always_on
description: `Rtinycc` is the core TinyCC and FFI package.
---

# Repo Scope And Rules

## Scope

`Rtinycc` is the core TinyCC and FFI package.

This repo is responsible for:

- bundled TinyCC build, configure, and installation logic
- CLI and low-level libtcc state management
- declarative FFI compilation and binding
- pointer, struct, enum, callback, and runtime helper APIs
- package-level semantics focused on clarity, soundness, and CRAN readiness

This repo is not the home for the experimental `tcc_quick()` compiler
front-end. That work belongs in the downstream `tccquickr` repo.

## Rules

- Never manually write `.Rd` files.
- Generate `.Rd` files from source documentation using `roxygen2`.
- `NEWS.md`, release notes, and other user-facing changelog text must stay
  user-oriented. Do not reference `AGENTS.md` or other internal agent/contributor
  guidance from user-facing documentation.

## Protection And Copying Model

Treat protection, copying, and ownership as part of the package semantics, not
as an implementation detail.

### Generated wrapper inputs

- Scalar inputs such as `i32`, `f64`, and `bool` are copied/coerced into C
  locals inside generated wrappers. They are not zero-copy views into R memory.
- `raw`, `integer_array`, `numeric_array`, and `logical_array` are the main
  zero-copy input paths. Generated wrappers pass direct pointers into the
  backing R vectors.
- This zero-copy path is only sound because the wrapper currently does not
  allocate fresh R objects between borrowing the vector storage and calling the
  target C function. If you add allocations in that window, re-audit GC safety.
- `cstring` input is not an owned copy. The wrapper uses
  `Rf_translateCharUTF8()` and passes the resulting pointer through for the
  duration of the call.
- `cstring_array` allocates a temporary `const char **` with `R_alloc()` on
  every call, then fills it element-by-element from translated R strings. This
  is a copied pointer array, not a deep copy of every string payload.
- `character_array` is not a `char **` interface. It currently exposes
  `STRING_PTR(x)`, meaning a vector of `SEXP` string cells. Do not document or
  reason about it as an array of C strings.
- `sexp` is the least transformed boundary mode: the wrapper passes the `SEXP`
  through directly.

### Generated wrapper returns

- Scalar returns are boxed into fresh R objects with `ScalarInteger`,
  `ScalarReal`, `ScalarLogical`, and related constructors.
- `cstring` returns are copied into R-managed string memory with `mkString()`.
  They are not retained as borrowed pointers.
- `ptr` returns stay as raw external pointers. No pointee copy is made and no
  ownership is implied.
- Array returns always allocate a fresh R vector and `memcpy()` the returned C
  buffer into it.
- When an array return uses `free = TRUE`, the generated wrapper frees the
  original C buffer after copying. This is still a copy path, not transfer of
  the original storage into R.

### Pointer helpers

- `tcc_malloc()` and `tcc_cstring()` create owned heap allocations tagged
  `rtinycc_owned` and backed by a finalizer.
- `tcc_data_ptr()` returns a borrowed pointer tagged `rtinycc_borrowed`.
- `tcc_read_ptr()` copies only the pointer-sized address value out of native
  memory (effectively `memcpy()` into a local `void *`), then wraps that
  address as a borrowed `externalptr`. It does not copy the pointee storage,
  does not infer ownership, and must not free whatever the address points to.
  If the returned wrapper is GC'd, that only reclaims the R `EXTPTRSXP`
  wrapper object, not the pointee.
- `tcc_read_bytes()` and `tcc_read_cstring_n()` copy data into fresh R objects.
- Typed scalar reads copy bytes from the pointed memory into local C scalars
  with `memcpy()`, then box those values for R.
- `tcc_free()` must stay strict: it only frees `rtinycc_owned` pointers and must
  refuse borrowed, struct, and other tagged pointers.

### Struct, union, and helper pointers

- Struct and union constructors allocate owned native storage and wrap it in an
  external pointer with a type-specific tag and finalizer.
- Field-address helpers return borrowed external pointers. The owner object is
  attached as the protected value so the pointee storage stays alive while the
  borrowed view exists.
- Raw struct read/write helpers are explicit copy operations over the struct
  byte buffer.

### Callback protection and copying

- Callback registry entries preserve the underlying R function with
  `R_PreserveObject()`. `tcc_callback_close()` is recommended because it
  deterministically releases that preserved function and invalidates the token.
- Synchronous callback trampolines allocate an R `VECSXP` argument list, fill
  it, call back into R, then convert the returned `SEXP` back to the declared C
  return type.
- Async callbacks necessarily copy more:
  - arguments are marshaled into heap-backed task structures
  - `cstring` async arguments are duplicated with `strdup()` for cross-thread
    safety
  - the main thread reconstructs fresh R objects from that task payload before
    invoking the callback
- Callback error defaults are NA-like sentinels or null pointers, not simple
  `0`/`FALSE` defaults. Do not document them loosely.

### Protection discipline

- Keep `PROTECT`/`UNPROTECT` balanced in both handwritten C and generated C.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sounkou-bioinfo/Rtinycc](https://github.com/sounkou-bioinfo/Rtinycc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
