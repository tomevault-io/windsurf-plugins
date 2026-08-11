---
trigger: always_on
description: > This document is designed to be read by both AI coding agents and human developers.
---

# 🤖 Agent Guidelines

> This document is designed to be read by both AI coding agents and human developers.
> It defines how to work on libdrmtap in a way that is clear, predictable, and reviewable.

---

## Quick Context for Agents

**What is libdrmtap?** A C library for capturing the Linux screen by reading the active scanout framebuffer directly via DRM/KMS. No Wayland portal, no PipeWire, no consent dialog — works under X11, Wayland, headless, and VMs.

**Key files to read first:**
- [`docs/research/05_api_and_architecture.md`](docs/research/05_api_and_architecture.md) — API design and architecture
- [`docs/research/02_drm_kms_mechanism.md`](docs/research/02_drm_kms_mechanism.md) — How DRM/KMS capture works
- [`docs/research/08_reframe_egl_analysis.md`](docs/research/08_reframe_egl_analysis.md) — EGL GPU-universal detiling (the primary detile path)
- [`docs/research/06_github_issues_analysis.md`](docs/research/06_github_issues_analysis.md) — Known gotchas

**Build system:** Meson  
**Language:** C11  
**License:** MIT  
**Tests:** unit (no hardware) + integration via VKMS (Virtual KMS) or a real GPU  
**Version:** ONE shared line across the C library, meson, `libdrmtap-sys` and `libdrmtap` (since 0.5.1). The canonical number lives in `include/drmtap.h` (`DRMTAP_VERSION_*`); `tools/check-version.sh` verifies every site. Deliberately not repeated here, because a copy of it goes stale on every release  

---

## Code Style

### C Style Rules

```c
// ✅ CORRECT: snake_case, 4 spaces, braces on same line
int drmtap_grab_mapped(drmtap_ctx *ctx, drmtap_frame_info *frame) {
    if (!ctx || !frame) {
        return -EINVAL;
    }
    
    int ret = refresh_plane_state(ctx);
    if (ret < 0) {
        set_error(ctx, "Failed to refresh plane: %s", strerror(-ret));
        return ret;
    }
    
    return 0;
}

// ❌ WRONG: camelCase, tabs, K&R braces
int drmtapGrabMapped(DrmtapCtx *ctx, DrmtapFrameInfo *frame)
{
	if (!ctx || !frame) return -EINVAL;
}
```

| Rule | Convention |
|---|---|
| Indent | 4 spaces, never tabs |
| Naming: functions | `snake_case`, prefixed `drmtap_` for public API |
| Naming: variables | `snake_case` |
| Naming: macros/constants | `UPPER_SNAKE_CASE`, prefixed `DRMTAP_` |
| Naming: types | `snake_case_t` for internal, `drmtap_*` for public |
| Braces | Same line (1TBS style) |
| Line length | 100 chars soft limit, 120 hard limit |
| Comments | `//` for single-line, `/* */` for multi-line |
| Header guards | `#ifndef DRMTAP_MODULE_H` / `#define DRMTAP_MODULE_H` |
| Error handling | Return negative errno values, never abort |
| Memory | Always check malloc returns, always free in cleanup |

### File Organization

```
// Every .c and .h file MUST start with this header block:

/*
 * libdrmtap — DRM/KMS screen capture library for Linux
 * https://github.com/fxd0h/libdrmtap
 *
 * Copyright (c) 2026 Mariano Abad <weimaraner@gmail.com>
 * SPDX-License-Identifier: MIT
 */

/**
 * @file drm_enumerate.c
 * @brief Plane, CRTC, and connector enumeration via DRM/KMS
 */

// After the header block, the file follows this structure:

// 1. Includes (grouped: system, library, project)
#include <stdio.h>        // system
#include <xf86drm.h>      // library
#include "drmtap.h"       // project

// 2. Private types and constants
#define MAX_PLANES 32

// 3. Static (private) functions
static int find_primary_plane(int drm_fd, uint32_t *plane_id) { ... }

// 4. Public functions (matching header declaration order)
int drmtap_list_displays(drmtap_ctx *ctx, ...) { ... }
```

### Doxygen Documentation

```c
// Public API functions in drmtap.h use Doxygen comments:

/**
 * @brief Capture a frame with mapped pixel data.
 *
 * Returns a pointer to linear RGBA pixel data in frame->data.
 * Handles GPU tiling → linear conversion automatically.
 *
 * @param ctx   Capture context from drmtap_open()
 * @param frame Output frame info (caller-allocated)
 * @return 0 on success, negative errno on error
 * @retval -EACCES Helper not found or not configured
 * @retval -ENODEV Display disconnected or CRTC inactive
 */
int drmtap_grab_mapped(drmtap_ctx *ctx, drmtap_frame_info *frame);

// Internal static functions use simple // comments:
// Refresh plane fb_id from the kernel (never cache!)
static int refresh_plane_state(drmtap_ctx *ctx) { ... }
```

### Commit Messages

```
component: short description (imperative mood)

Longer explanation of what and why (not how).
Reference issues with #NNN.

Examples:
  grab: refresh plane fb_id on every frame
  helper: export scanout as DMA-BUF and pass fd via SCM_RIGHTS
  gpu-egl: detile CCS framebuffers via EGLImage import
  tests: add vkms enumeration test
  docs: update GPU compatibility table
```

Prefixes: `grab`, `enumerate`, `formats`, `cursor`, `helper`, `gpu-egl`, `gpu-intel`, `gpu-amd`, `gpu-nvidia`, `gpu-generic`, `tests`, `docs`, `build`, `ci`.

---

## Success Criteria

Every change must satisfy ALL of these before merge:

### 1. It compiles
```bash
meson setup build && meson compile -C build
# Zero warnings with -Wall -Wextra -Werror
```

### 2. It doesn't break existing tests
```bash
meson test -C build
# All existing tests pass
```

### 3. New code has tests (when possible)
- Any new function should have a corresponding test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fxd0h/libdrmtap](https://github.com/fxd0h/libdrmtap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
