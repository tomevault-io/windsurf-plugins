---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

This project is written in the [Odin programming language](https://odin-lang.org/).

**Build (Makefile):**

```bash
make debug    # Debug build → build/debug (-define:VERBOSE_DEBUG=1)
make release  # Release build → build/release (-o:speed -no-bounds-check -define:PROFILING_ENABLED=false -define:VERBOSE_OUTPUT=false -define:TRACE_CAPTURE_ENABLED=false)
```

**Manual build:**

```bash
# Debug
odin build . -collection:RT_Weekend=. -debug -define:VERBOSE_DEBUG=1 -out:build/debug

# Release (optimized, quiet stdout)
odin build . -collection:RT_Weekend=. -o:speed -no-bounds-check -define:PROFILING_ENABLED=false -define:VERBOSE_OUTPUT=false -define:TRACE_CAPTURE_ENABLED=false -out:build/release
```

**Run:**

```bash
# Run with defaults (800px wide, 10 samples, 10 spheres)
./build/debug
# or
./build/release

# Custom resolution / samples / threads
./build/debug -w 400 -h 225 -s 20 -c 8

# Full options
./build/debug -w <int> -h <int> -s <samples> -n <spheres> -c <threads>
```

**VERBOSE_DEBUG:** Convenience compile-time flag (`#config`, default `0`). Use `-define:VERBOSE_DEBUG=1` to enable diagnostics defaults in one switch:
- `TRACK_ALLOCATIONS` default becomes enabled
- `UI_EVENT_LOG_ENABLED` default becomes enabled
- `EDITOR_IDLE_GPU_TRACE` default becomes enabled

Individual flags can still be overridden explicitly with their own `-define`.  
Runtime override for idle GPU tracing: `EDITOR_IDLE_GPU_TRACE=0|1` (`0/off/false` disables, `1/on/true` enables).

**VERBOSE_OUTPUT:** A compile-time flag (`#config`, default `true`) controls non-essential stdout: startup config, system info, per-thread stats, timing breakdown, and GPU success messages. Error and fallback messages (e.g. GPU init failed) are always printed. Release builds set `VERBOSE_OUTPUT=false`.

**TRACE_CAPTURE_ENABLED:** When `true`, Chrome-trace visual benchmarking (Render → Start/Stop Visual Benchmark Capture) is available; when `false`, trace code is compiled out and the benchmark menu items are disabled. Release builds set `TRACE_CAPTURE_ENABLED=false` so capture and file I/O add no overhead.

**FILE_MODAL_FALLBACK:** When `true`, if the native file dialog is unavailable (e.g. zenity not installed), the editor falls back to the text path modal for Import and Save As. Default is `false`. Defined in `editor/core_cmd_actions.odin`.

**Key flags:**
- `-w` / `-h`: image width / height in pixels
- `-s`: samples per pixel (default 10)
- `-n`: number of small random spheres (default 10)
- `-c`: thread count (defaults to physical CPU cores)

The program opens a **Raylib window** with floating panels:
- **Viewport** — unified Editor / Raytrace view with mode toggle and render settings
- **Stats** — tile progress, thread count, elapsed time
- **Console** — startup and completion messages
- **Details** — selected object properties
- **World Outliner** — scene object list

Drag panels by their title bar; resize from the bottom-right grip.

## Dependencies

### Raylib (via Odin vendor collection)
Raylib is bundled with the Odin compiler distribution under `$(odin root)/vendor/raylib/`.
The Linux shared library (`libraylib.so`) is included in `vendor/raylib/linux/` — no separate system install is required as long as your Odin installation is complete.

If you encounter linker errors on Linux, verify that:
```bash
ls $(odin root)/vendor/raylib/linux/libraylib.so
```
exists. If not, install Odin from [odin-lang.org](https://odin-lang.org/) with the full vendor collection.

### GPU path (Vulkan compute)
The GPU compute-shader backend uses **Vulkan** via the `vk_ctx` package (`vendor:vulkan` + `vendor:glfw`). On platforms without a Vulkan driver, `create_gpu_renderer` returns nil and the renderer falls back to CPU automatically.

### Vulkan infrastructure (`vk_ctx`)
The **`vk_ctx`** package bootstraps a Vulkan instance, device, queues, command pools, compute pipeline, buffer management, and synchronization utilities. It is used by the GPU backend (`raytrace/gpu_backend_vulkan.odin`) and the standalone smoke test. Build the smoke binary from the repo root:

```bash
make vk-smoke
./build/vk_smoke --headless          # instance / device / queues / pool
./build/vk_smoke --headless --clear  # + one-shot clear-color image submit
./build/vk_smoke --window [--platform=auto|x11|wayland] [--frames=N] # present Vulkan hello-triangle
```

Requires a system **Vulkan loader** (`libvulkan`) and **GLFW** (Odin’s `vendor/glfw` links the system or static library). `vk_load_vulkan` checks `glfw.VulkanSupported()`, that `vkGetInstanceProcAddr` is non-null after loading, and that `vkEnumerateInstanceExtensionProperties` succeeds, so missing loaders/ICDs fail with an error instead of crashing later.

If the loader advertises **VK_KHR_portability_enumeration**, `create_instance` enables it and sets **VK_INSTANCE_CREATE_ENUMERATE_PORTABILITY_BIT_KHR** so portability ICDs (e.g. MoltenVK) are enumerated; **VK_KHR_portability_subset** is enabled on the logical device when the GPU supports it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThadeuFerreira) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
