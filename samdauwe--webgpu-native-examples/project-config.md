---
trigger: always_on
description: This file defines the mandatory coding standards for all C99 WebGPU examples in this repository.
---

# CLAUDE.md — Repository Coding Standards

This file defines the mandatory coding standards for all C99 WebGPU examples in this repository.

## Project Layout

```
src/examples/   — Example source files (one .c file per example)
src/core/       — Shared core utilities (camera.c, gltf_model.c, etc.)
src/webgpu/     — WebGPU helpers (wgpu_common.h / wgpu_common.c)
assets/         — Textures, models, fonts, shaders
wasm/           — WebAssembly CMake build
```

## Required Header

Every example **must** use the functional header:

```c
#include "webgpu/wgpu_common.h"
```

## Example Structure

Follow the pattern established in `src/examples/two_cubes.c` and `src/examples/normal_map.c`:

### 1. Global State Struct

All example state lives in a single global struct:

```c
static struct {
  WGPURenderPipeline pipeline;
  WGPUBuffer vertex_buffer;
  /* ... */
} state = {0};
```

### 2. Function Naming

Use `init_` prefix:

```c
static void init_pipeline(wgpu_context_t* wgpu_context) { ... }
static void init_buffers(wgpu_context_t* wgpu_context) { ... }
```

### 3. File Organization

```
[includes]
[shader variable declarations]   ← declare at top of file
[state struct]
[init_ functions]
[update_ functions]
[frame() function]
[input_event_cb callback]
[main() function]
[shader code using CODE() macro] ← actual WGSL at bottom of file
```

### 4. main() Function

Model `main()` after `src/examples/normal_map.c`:

```c
int main(int argc, char* argv[]) {
  wgpu_desc_t desc = {
    .title = "Example Title",
    .width = 1280,
    .height = 720,
    .init_cb    = init,
    .frame_cb   = frame,
    .shutdown_cb = shutdown,
    .input_event_cb = input_event_cb,
  };
  return wgpu_start(&desc);
}
```

## Shader Code

- Declare shader string variables at the **top** of the file (before the state struct).
- Place the **actual WGSL code** at the **bottom** of the file using the `CODE()` macro.
- Disable clang-format around shader code:

```c
// clang-format off
static const char* my_shader_wgsl = CODE(
  struct Uniforms {
    mvp : mat4x4<f32>,
  }
  @group(0) @binding(0) var<uniform> uniforms : Uniforms;
  /* ... */
);
// clang-format on
```

- Translate **GLSL → WGSL** (no raw GLSL in WebGPU examples).
- Split long shaders into `_part1` / `_part2` chunks — ISO C99 limits string literals to 4095 characters.

## Input Handling

- Use `input_event_cb` for all input (mouse, keyboard, resize). Reference: `src/examples/cameras.c`.
- Camera handling uses `src/core/camera.c`.
- When the GUI is active, do **not** forward mouse/keyboard events to the 3D scene:

```c
static void input_event_cb(const wgpu_input_event_t* event) {
  if (wgpu_imgui_want_capture_mouse()) return; /* GUI owns this event */
  /* handle 3D scene input */
}
```

## Asynchronous File Loading

Use `sokol_fetch` for all asset loading. Reference: `src/examples/textured_cube.c`.

- Never hard-code a fixed file buffer size as a static array — allocate dynamically using the file size (e.g. `stat()`) to avoid bloated executables.
- Always free fetch buffers in the callback after upload to GPU.
- Log errors on fetch failure.

```c
sfetch_send(&(sfetch_request_t){
  .path      = "assets/textures/my_texture.png",
  .callback  = texture_fetch_callback,
  .buffer    = { .ptr = buf, .size = buf_size },
});
```

## GUI (cimgui)

Add GUI support via the cimgui library. Reference: `src/examples/normal_map.c`.

- Initialize: `wgpu_imgui_init(wgpu_context)`
- Render in `frame()`: `wgpu_imgui_new_frame()` … `wgpu_imgui_render()`
- Shutdown: `wgpu_imgui_shutdown()`
- Store all GUI-controlled parameters in the global state struct.
- Check `wgpu_imgui_want_capture_mouse()` / `wgpu_imgui_want_capture_keyboard()` before processing scene input.

## Allowed External Libraries

| Library | Purpose |
|---|---|
| `cglm` | Math (vectors, matrices, quaternions) |
| `cgltf` | glTF/GLB model loading |
| `cJSON` | JSON parsing |
| `stb_image` | Image loading (PNG, JPG, HDR, …) |
| `sokol_fetch` | Asynchronous file loading |
| `sokol_time` | Timing / animation |
| `sokol_log` | Logging |

Do **not** add other runtime dependencies without strong justification.

## Memory & Performance Rules

1. **No hot-path allocations** — allocate once during init, free during shutdown.
2. **No memory leaks** — every `malloc`/`wgpuXxxCreate` must have a matching `free`/`wgpuXxxRelease` in the shutdown path.
3. **Resize safety** — recreate only swap-chain-dependent resources (depth texture, render targets) on resize; do not recreate pipelines or static buffers.
4. **No runtime validation errors** — the window must be resizable without WebGPU validation errors.
5. Prefer fixed-length stack arrays over heap allocation for small, known-size data.

## Naming Conventions

| Kind | Style | Example |
|---|---|---|
| Functions | `snake_case` | `init_pipeline`, `update_uniforms` |
| Types / structs | `snake_case` with `_t` suffix | `wgpu_context_t`, `camera_t` |
| Constants / `#define` | `SCREAMING_SNAKE_CASE` | `MAX_LIGHTS`, `BUFFER_SIZE` |
| Local variables | `snake_case` | `vertex_count`, `bind_group` |
| Global state struct | unnamed `static struct { … } state` | see §1 above |
| Shader variables | `<name>_wgsl` or `<name>_wgsl_part1/2` | `vertex_shader_wgsl` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samdauwe/webgpu-native-examples](https://github.com/samdauwe/webgpu-native-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
