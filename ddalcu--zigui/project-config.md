---
trigger: always_on
description: Agent-facing guide: how to build/test, the non-obvious gotchas, the architecture
---

# CLAUDE.md — working on zigui

Agent-facing guide: how to build/test, the non-obvious gotchas, the architecture
map, the recipe for adding a component, and how the post-v0 feature set
(navigation, tabs, modals, grids, animation, materials, accessibility, HiDPI) is
implemented. User-facing docs live in [README.md](README.md); design rationale in
[PRD.md](PRD.md).

## TL;DR

`zigui` is a SwiftUI-like UI library in **pure Zig 0.16**. The core (geometry,
color, state, layout, theme, text, canvas, software rasterizer, GPU scene
translation, view layer, components) has **no C dependencies** and is fully
unit-tested headlessly. SDL3 links only into example executables (`src/app.zig`
+ `src/gpu/gpu.zig`). On screen, rendering is **GPU-accelerated via SDL_GPU**
(Metal on macOS, Vulkan on Linux/Windows) with an automatic fallback to the
**CPU software rasterizer** presented through an SDL streaming texture — both
backends consume the same `Canvas` command list and produce output identical to
within 1 LSB, so tests/CI stay on the software path. See
[GPU backend](#gpu-backend--sdl_gpu-metalvulkan-with-software-fallback).

The post-v0 feature set — grids, tabs, navigation, modals/overlays, animation,
materials/blur, accessibility, and HiDPI — is **implemented**; see
[Post-v0 features](#post-v0-features-implemented) for the shipped APIs and the
seams each one uses.

## Build / test / run — and the gotchas

```sh
zig build test --summary all                # 169 tests, headless. THIS is the inner loop.
zig build showcase edit                      # build the two examples (does NOT run them)
zig build run-showcase                       # opens a window — blocks on the event loop
docker build -t zigui-test .                 # run the full suite on Linux

# Headless UI iteration (renders one frame to a BMP, no window):
./zig-out/bin/showcase --screenshot /tmp/sc.bmp [section] [--dark] [--accent N]
./zig-out/bin/edit --screenshot /tmp/edit.bmp [file] [--demo-find|--demo-dialog]
# sips -s format png /tmp/sc.bmp --out /tmp/sc.png   # to view on macOS
# --bench N re-rasterizes the frame N times and prints ms/frame (build with
# -Doptimize=ReleaseFast) — use it when touching src/render/raster.zig.
# --gpu (showcase only) renders the frame through the real SDL_GPU pipeline
# offscreen (works headlessly on macOS/Metal) — THE verification loop when
# touching src/gpu/ or src/render/gpu_scene.zig; diff the BMP against the
# software output. Combine with --bench N for a full-sync GPU ms/frame.
```

There are exactly **two examples**: `examples/showcase` (the kitchen-sink gallery
— every public component across sidebar sections, plus live light/dark and accent
switchers, built on the macOS 26 theme) and `examples/edit` (the multi-line text
editor). The old `hello`/`settings`/`llm-chat`/`screenshot` examples were folded
into `showcase` and removed.

- **Use `zig build test`, NOT `zig test src/zigui.zig`.** The bundled Inter font
  is an anonymous import `inter_font` wired only in `build.zig`
  (`mod.addAnonymousImport("inter_font", ...)`); `text/ttf.zig` does
  `@embedFile("inter_font")`. The raw `zig test` invocation has no such import
  and fails.
- **Never run `zig build run-*` in a headless/agent context** — it opens an SDL
  window and blocks in `SDL_WaitEvent`. Use `zig build showcase edit` to verify
  the backend compiles/links, or the `--screenshot` flag to render one frame.
- Tests are **inline** (`test "..." {}` blocks). A module's tests only run if the
  root (`src/zigui.zig`) imports it — add a `pub const x = @import(...)` there.
- **Zig 0.16 gotchas hit during the build** (so you don't rediscover them):
  - `std.ArrayList(T)` is **unmanaged**: `var l: std.ArrayList(T) = .empty;`
    then `l.append(allocator, x)`, `l.deinit(allocator)`.
  - `std.testing.refAllDeclsRecursive` is gone; use `refAllDecls`.
  - `std.fs` is reorganized (no `cwd()`/`accessAbsolute` free fns; moved to
    `std.Io.Dir`). The build script avoids fs probing entirely.
  - **The new `std.Io` model removed a lot from `std`**: the socket primitives
    (`socket`/`connect`/`fcntl`/`send`/`recv`/`close`) are gone from `std.posix`
    *and* not `pub` in `std.c`; `std.net` moved to `std.Io.net`; `std.time` lost
    `milliTimestamp`/`sleep`; `std.process.argsAlloc` is gone (args arrive via a
    `std.process.Init.Minimal` first param to `main`, iterated with
    `std.process.Args.iterate`); `std.heap.GeneralPurposeAllocator` is gone (use
    `page_allocator`/`DebugAllocator`). `examples/llm-chat` sidesteps the socket
    gap by `@cImport`-ing the POSIX headers (libc is linked) — see its
    `chat_client.zig`.
  - **Inferred error sets + recursion = "dependency loop"**. Mutually/​self-
    recursive fns that allocate must declare an explicit error set, e.g.
    `Allocator.Error!void` (see `engine.arrange`, `ttf.collectEdges`).
  - **Don't name fields/methods after primitives** (`u16`, `i16`, …) — compile
    error. (See the `Be` reader in `ttf.zig` using `rdU16` etc.)

## Architecture map (data flow)

```
View (value tree)               src/view/view.zig
  └─ buildNode → layout.Node     (lowering; modifiers become padding/frame nodes)
       └─ engine.arrange → LayoutResult (frames)   src/layout/engine.zig (+ stack.zig)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddalcu/zigui](https://github.com/ddalcu/zigui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
