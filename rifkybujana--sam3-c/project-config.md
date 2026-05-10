---
trigger: always_on
description: SAM3 is a pure C11 inference engine for Facebook's Segment Anything Model 3.
---

# SAM3 — Pure C Inference Engine

## What This Project Is

SAM3 is a pure C11 inference engine for Facebook's Segment Anything Model 3.
Metal backend first, extensible to CUDA/Vulkan. Modeled after ggml/llama.cpp.
Upstream reference: https://github.com/facebookresearch/sam3

### CRITICAL: This is SAM3, not SAM2

- **Always refer to this project as SAM3** (Segment Anything Model 3).
- **Never use "SAM2", "sam2", or "Segment Anything Model 2"** in code,
  comments, commit messages, or conversation. The upstream repo is `sam3`.
- All prefixes are `sam3_`, all macros are `SAM3_`, all paths use `sam3/`.
- If referencing the upstream Python repo, it is `facebookresearch/sam3`.

## Build

    mkdir build && cd build
    cmake .. -DCMAKE_BUILD_TYPE=Debug
    make -j$(nproc)

Run tests:

    cd build && ctest --output-on-failure

## Directory Map

    include/sam3/     Public API headers (sam3.h, sam3_types.h)
    src/core/         Tensor ops, arena allocator, compute graph
    src/backend/      Backend abstraction + Metal/CPU implementations
    src/model/        SAM3 layers (image encoder, prompt encoder, mask decoder)
    src/util/         Logging, error codes
    tools/            CLI binaries (inference, weight conversion)
    tests/            Unit and integration tests
    models/           Model weights (.gitignored)
    docs/             Documentation (format specs, reference material)

## .sam3 Weight Format

Model weights are stored in the `.sam3` binary format. See
[docs/weight-format.md](docs/weight-format.md) for the full specification.

Quick summary:
- **Magic**: `0x334D4153` (ASCII "SAM3", little-endian)
- **Version**: 2
- **Layout**: 48-byte header + tensor descriptors (176 B each) + page-aligned
  data blob
- **Alignment**: data blob at 4096-byte boundary, tensors at 64-byte boundaries
- **Loading**: mmap-based with FNV-1a hash table for O(1) tensor lookup
- **Dtypes**: F32, F16, BF16, I32, I8, Q8_0 (block-quantized int8)
- **Conversion**: `sam3_convert` reads SafeTensors, writes `.sam3`
- **Source**: `src/core/weight.h` (format structs), `src/core/weight.c`
  (loader/writer)

## C Coding Standard

These rules are non-negotiable. Every file must follow them exactly.

### Language

- **C11 only.** No C++ features, no GNU extensions unless guarded by `#ifdef`.
- Compile with: `-std=c11 -Wall -Wextra -Wpedantic`
- Debug builds add: `-Werror -fsanitize=address,undefined`

### Formatting

- **Tabs for indentation, 8 characters wide.** This is the Linux kernel convention.
  It forces you to keep nesting shallow.
- **80-column soft limit, 100 hard limit.** Break long lines at operators or after commas.
- **K&R brace style** for functions (opening brace on its own line).
  Same-line braces for `if`, `for`, `while`, `switch`, `struct`.

```c
/* Function: opening brace on its own line */
static int compute_offset(int row, int col, int stride)
{
	if (row < 0 || col < 0) {
		return -1;
	}

	return row * stride + col;
}
```

### Section Comments

For dividing a file into labeled sections, use a single-line comment with
short dash runs around the label. No multi-line dashed blocks.

Good:

```c
/* --- sam3_frame_cache_init --- */
```

Bad (do not use):

```c
/* ------------------------------------------------------------------ */
/* sam3_frame_cache_init                                               */
/* ------------------------------------------------------------------ */
```

### Naming

- **`snake_case` for everything:** functions, variables, types, enum values, macros.
- **Prefix public symbols with `sam3_`.**
  Internal symbols use subsystem prefix: `tensor_`, `metal_`, `graph_`, etc.
- **No Hungarian notation.** No `pFoo`, `m_bar`, `szName`.
- **No typedef hiding pointers.** If it's a pointer, the `*` must be visible.
- Typedefs are acceptable for opaque structs in public API:
  `typedef struct sam3_ctx sam3_ctx;`

### File Documentation Header

**Every `.c` and `.h` file MUST begin with this header.** This is the single most
important convention — it gives an LLM instant context about any file.

```c
/*
 * <relative/path/to/file> - <one-line description>
 *
 * <2-4 sentences explaining purpose, role in the system, and key design
 * decisions. Mention what subsystem this belongs to and how it fits into
 * the larger architecture.>
 *
 * Key types:  <primary structs/enums defined or used here>
 * Depends on: <direct header dependencies, not transitive>
 * Used by:    <files that directly include or call into this>
 *
 * Copyright (c) 2026 Rifky Bujana Bisri
 * SPDX-License-Identifier: MIT
 */
```

Rules for the header:
- `Key types` lists the 1-3 most important types. Not every type.
- `Depends on` lists headers this file directly includes (not system headers).
- `Used by` lists files that directly depend on this one. Update when adding
  new callers. "Unknown" is acceptable for new files.
- Keep the description factual. No aspirational language.

### Function Documentation

Document non-trivial functions with a comment block above:

```c
/*
 * sam3_tensor_reshape - Change tensor dimensions without copying data.
 *
 * @t:        Tensor to reshape (must not be a view)
 * @new_dims: Array of new dimension sizes
 * @n_dims:   Number of dimensions (1-4)
 *

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rifkybujana/sam3.c](https://github.com/rifkybujana/sam3.c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
