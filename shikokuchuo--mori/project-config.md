---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

mori — Shared Memory for R Objects. Uses POSIX shared memory (Linux, macOS) and Win32 file mappings (Windows) with R's ALTREP framework to let multiple processes on the same machine read the same physical memory pages. No external dependencies. Requires R >= 4.3.0 (for ALTLIST). API: `share()` → ALTREP shared object, `map_shared()` → open SHM by name, `shared_name()` → extract SHM name, `is_shared()` → test if shared. SHM lifetime is automatic — managed by R's garbage collector via chained external pointer finalizers. ALTREP serialization hooks serialize standalone shared objects as the SHM name (~30 bytes) and nested ALTLIST elements (including sub-lists) as `(parent_name, integer_path)`, enabling transparent use with mirai and any R serialization path. Nested VECSXP elements are zero-copy: each level is an inline self-describing MORL region inside the parent SHM.

## Development Commands

### Testing

```r
# Run the full testthat suite
devtools::test()
```

### Building and Checking

```bash
# Build package
R CMD build .

# Check package
R CMD check --no-manual mori_*.tar.gz
```

```r
# Generate documentation from roxygen2 comments
devtools::document()
```

## Key Architecture

### Storage Model

- **Zero-copy (SHM-backed)**: All atomic vectors (including character vectors and those with arbitrary attributes such as names, class, levels, dim) and data frame columns are written directly into SHM and backed by ALTREP on consumers. Attributes are serialized into a trailing section of the SHM region and restored via `SET_ATTRIB` on the consumer. For numeric types, `Dataptr_or_null` returns the SHM pointer for reads; `Dataptr(writable=TRUE)` materializes a private copy (COW). For character vectors, `Elt` lazily creates each CHARSXP via `Rf_mkCharLenCE` from the SHM data; `Dataptr_or_null` returns NULL to force element-by-element access.
- **Nested lists (zero-copy)**: VECSXP/LISTSXP elements of a shared list are written inline as a complete child MORL region at their `data_offset`. Each nested region is self-describing (magic bytes + header) and recursively contains its own directory, element data, and attributes. Consumers wrap each level in its own ALTLIST via a lightweight `mori_list_view` (pointer + bounds + index) rather than a full `mori_shm`. Sub-lists therefore stay zero-copy all the way down and report `is_shared(x) == TRUE`; `shared_name()` returns the root SHM name for root lists and `""` for sub-lists.
- **Pass-through**: All other R objects (environments, closures, language objects) are returned unchanged by `share()`. No SHM is created.

### share() Dispatch Logic (altrep.c: `mori_create`)

All R exported functions are single `.Call` wrappers. `share()` calls `mori_create` which dispatches on `TYPEOF(x)`:

0. If `x` is already a mori-backed ALTREP (detected via the `mori_owned_tag` on `R_altrep_data1(x)`), return `x` unchanged. This makes `share()` idempotent and also short-circuits re-sharing of sub-list views and element vectors extracted from an ALTLIST.
1. `NILSXP` → returned as-is (falls through all checks).
2. `VECSXP`/`LISTSXP` → `mori_shm_create_list_call` — ALTLIST with per-element directory. Writing is split into two passes: `mori_nested_size` computes the total SHM size (recursing into nested VECSXP/LISTSXP elements), and `mori_nested_write` produces a complete MORL region at the target base pointer, recursing into child VECSXP/LISTSXP elements by calling itself on `base + child_data_offset`. Each element is independently zero-copy (any atomic, with or without attributes), serialized as bytes, or a nested MORL region. Data frames and pairlists go through this path (pairlists are coerced to VECSXP via `Rf_coerceVector`, both at the top level and at each level of the recursion).
3. `STRSXP` → `mori_shm_create_string_call` — ALTSTRING backed by SHM with offset table + packed string data. Attributes (if any) are serialized after the string data.
4. Other SHM-eligible types (`REALSXP`, `INTSXP`, `LGLSXP`, `RAWSXP`, `CPLXSXP`) → `mori_shm_create_vector_call` — single ALTREP vector backed by SHM. Attributes (if any) are serialized after the vector data.
5. Everything else → returned as-is (pass-through).

Each creation path returns the ALTREP result via `mori_make_result`, which chains the host extptr (responsible for `shm_unlink`) into the ALTREP's external pointer hierarchy. SHM lifetime is thus tied to the R object — when the ALTREP is garbage collected, both munmap and unlink happen automatically.

### ALTREP Serialization Hooks

All ALTREP classes register `Serialized_state` and `Unserialize` methods.

- **Standalone shared objects** (created by `share()` or `map_shared()`) serialize as just the SHM name string (~30 bytes). On unserialize, `mori_Unserialize` validates the name via `mori_is_shm_name()` (checks `/mori_` prefix on POSIX, `Local\mori_` on Windows), then `mori_shm_open_and_wrap` opens the SHM and creates a fresh ALTREP wrapper. R's ALTREP serialization framework separately serializes and restores the object's attributes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shikokuchuo/mori](https://github.com/shikokuchuo/mori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
