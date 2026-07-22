---
trigger: always_on
description: This document provides guidelines and conventions for AI assistants (particularly Codex) when working on the Albucore codebase.
---

# Codex AI Development Guidelines for Albucore

This document provides guidelines and conventions for AI assistants (particularly Codex) when working on the Albucore codebase.

## Project Overview

Albucore is a high-performance image processing library that provides optimized atomic functions for image manipulation. It serves as the foundation for AlbumentationsX and focuses on:

- Maximum performance through multiple backend implementations (NumPy, OpenCV, custom)
- **Supported dtypes: uint8 and float32 only** (no float64)
- **Backend routing is based ONLY on performance** — benchmark before choosing. If LUT is slower, don't use it.
- Consistent API across different image types and shapes

## Core Documentation

Please familiarize yourself with these key documents:

1. **[Image Shape Conventions](docs/image-conventions.md)** - **CRITICAL** rules about image shapes
   - All images must have an explicit channel dimension
   - Grayscale images use shape `(H, W, 1)`, not `(H, W)`
   - Consistent dimension indexing: `shape[-1]` is channels, `shape[-2]` is width, `shape[-3]` is height

2. **[Decorators](docs/decorators.md)** - Decorator usage and patterns
   - `@preserve_channel_dim` - Maintains single-channel shape
   - `@contiguous` - Ensures C-contiguous memory layout
   - `@clipped` - Clips to valid dtype range
   - `@float32_io` / `@uint8_io` - Type conversion wrappers
   - `@batch_transform` - Batch processing patterns

3. **[Performance Optimization](docs/performance-optimization.md)** - Performance best practices
   - LUT operations and float32 dtype management
   - Backend selection strategies
   - Memory layout considerations
   - Benchmarking guidelines

4. **[Public API](docs/public-api.md)** - Public routers, star exports, and `albucore.functions` compatibility shims

5. **[NumKong Performance](docs/numkong-performance.md)** - NumKong vs OpenCV/NumPy/LUT benchmark tables and methodology

## Repo-Local Codex Skills

Repo-specific Codex skills live in `.codex/skills/`:

- `albucore-conventions` - Apply when implementing or reviewing image-processing code, tests, shape handling, dtype handling, and backend routing.
- `albucore-benchmarks` - Apply when adding benchmarks, comparing versions, or documenting benchmark workflow.
- `albucore-public-api` - Apply when changing exports, documenting API status, or deciding what belongs in package `__all__`.

Keep these skills aligned with `AGENTS.md` and the docs above when conventions change.

## Development Principles

### 1. Image Shape Convention

**This is the most important rule:**

```python
# ✅ ALWAYS use explicit channel dimension
grayscale_image = np.array(..., shape=(H, W, 1))

# ❌ NEVER use implicit channel dimension
grayscale_image = np.array(..., shape=(H, W))  # WRONG!
```

Given any image, you can always assume:
- `num_channels = image.shape[-1]`
- `width = image.shape[-2]`
- `height = image.shape[-3]`

### 2. Performance First — Routing by Benchmark Only

- **Choose implementations ONLY based on benchmark results.** No conventions. If LUT is slower for a given case, use something else.
- Always benchmark when adding new implementations or changing routing
- OpenCV, NumPy, LUT — pick whichever is fastest for the target dtype/shape

### 3. Type Safety

- Use type hints consistently
- Leverage the provided type aliases: `ImageType`, `ImageUInt8`, `ImageFloat32`, `ValueType`
- Ensure dtype preservation or explicit conversion

### 4. Multiple Implementations Pattern

Route based on **benchmarked performance**, not convention:

```python
@clipped
def operation(img: ImageType, value: ValueType, inplace: bool = False) -> ImageType:
    num_channels = get_num_channels(img)
    value = convert_value(value, num_channels)

    # Route based on what benchmarks show is fastest for this dtype/shape
    if img.dtype == np.uint8:
        return operation_lut(img, value, inplace)  # Only if LUT wins the benchmark

    if img.dtype == np.float32:
        return operation_numpy(img, value)  # Or opencv — whichever is faster

    raise ValueError(f"Unsupported dtype {img.dtype}. Albucore supports only uint8 and float32.")
```

### 5. Use Decorators

Albucore provides several useful decorators:

- `@preserve_channel_dim` - Maintains `(H, W, 1)` shape when OpenCV might drop it
- `@contiguous` - Ensures C-contiguous memory layout
- `@clipped` - Clips results to valid dtype range
- `@float32_io` / `@uint8_io` - Type conversion wrappers

### 6. Testing

- Write tests for uint8 and float32 only
- Test single images, batches, volumes, and batch of volumes
- Test edge cases: single-channel, many channels (>4), extreme values
- Include performance benchmarks when relevant

### 7. Dependency Lock Consistency

- When changing dependencies in `pyproject.toml`, update `uv.lock` in the same PR.
- Validate lock consistency with `uv lock --check`.
- Release flow uses `uv export --frozen`; stale `uv.lock` can break release artifact generation.

## Code Style

### Imports

```python
from collections.abc import Callable
from typing import Any, Literal

import cv2
import numpy as np
import numkong as nk
import stringzilla as sz

from albucore.decorators import contiguous, preserve_channel_dim

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [albumentations-team/albucore](https://github.com/albumentations-team/albucore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
