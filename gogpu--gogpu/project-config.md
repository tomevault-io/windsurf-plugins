---
trigger: always_on
description: > Pure Go GPU Computing Ecosystem. 1.1M+ LOC, zero CGO, cross-platform.
---

# AGENTS.md — GoGPU Ecosystem

> Pure Go GPU Computing Ecosystem. 1.1M+ LOC, zero CGO, cross-platform.

## What is GoGPU

GoGPU is a complete GPU computing ecosystem for Go — think Flutter or Qt, but Pure Go with zero CGO. Full stack from shader compilation and GPU abstraction to 2D/3D rendering and GUI widgets. Single binary deployment with `go build`.

Three backends via build tags — same API, same code:
- **Pure Go** (default) — zero dependencies, cross-compile anywhere
- **Rust FFI** (`-tags rust`) — battle-tested wgpu-native drivers via go-webgpu
- **Browser WASM** (`GOOS=js GOARCH=wasm`) — runs in browser via WebGPU API

Planned: Android (ANativeWindow) and iOS (CAMetalLayer) platform support.

## Ecosystem Map

| Package | Import | Purpose |
|---------|--------|---------|
| **gogpu** | `github.com/gogpu/gogpu` | App framework, windowing, input, lifecycle |
| **wgpu** | `github.com/gogpu/wgpu` | WebGPU implementation (Vulkan/Metal/DX12/GLES/Software) |
| **naga** | `github.com/gogpu/naga` | Shader compiler (WGSL → SPIR-V/MSL/GLSL/HLSL/DXIL) |
| **gg** | `github.com/gogpu/gg` | 2D graphics, text, paths, images |
| **ui** | `github.com/gogpu/ui` | GUI toolkit (22+ widgets, Material 3 / Fluent / Cupertino) |
| **gpucontext** | `github.com/gogpu/gpucontext` | Shared interfaces (DeviceProvider, EventSource) |
| **gputypes** | `github.com/gogpu/gputypes` | WebGPU type definitions |
| **systray** | `github.com/gogpu/systray` | System tray (Win32/macOS/Linux) |
| **audio** | `github.com/gogpu/audio` | Audio engine (WASAPI/CoreAudio/PulseAudio) |
| **g3d** | `github.com/gogpu/g3d` | 3D rendering, scene graph, PBR |
| **compose** | `github.com/gogpu/compose` | Multi-process composition (Unix socket IPC, LZ4) |
| **editor** | `github.com/gogpu/editor` | Text/Code editor widget (GPU-accelerated, early dev) |

## Which Package Do I Need?

- **Render a window with GPU** → `gogpu`
- **Draw 2D shapes, text, images** → `gg` + `gogpu`
- **Build a desktop GUI app** → `ui` + `gogpu`
- **Use WebGPU directly** → `wgpu`
- **Compile shaders** → `naga`
- **System tray icon** → `systray`
- **Play audio** → `audio`
- **3D rendering** → `g3d` + `gogpu`

## Quick Start

### Minimal window with triangle

```go
package main

import (
    "github.com/gogpu/gogpu"
    "github.com/gogpu/gogpu/gmath"
)

func main() {
    app := gogpu.NewApp(gogpu.DefaultConfig().
        WithTitle("Hello GoGPU").
        WithSize(800, 600))

    app.OnDraw(func(dc *gogpu.Context) {
        dc.DrawTriangleColor(gmath.DarkGray)
    })

    app.Run()
}
```

### 2D graphics (gg + gogpu)

```go
import (
    "github.com/gogpu/gg"
    "github.com/gogpu/gg/integration/ggcanvas"
    _ "github.com/gogpu/gg/gpu" // GPU acceleration
)

canvas, _ := ggcanvas.New(app.GPUContextProvider(), 800, 600)

app.OnDraw(func(dc *gogpu.Context) {
    canvas.Draw(func(cc *gg.Context) {
        cc.SetRGB(1, 0, 0)
        cc.DrawCircle(400, 300, 100)
        cc.Fill()
    })
    canvas.Render(dc.AsTextureDrawer())
})
```

### GUI application (ui + gogpu)

```go
import (
    "github.com/gogpu/gogpu"
    "github.com/gogpu/ui/app"
    "github.com/gogpu/ui/desktop"
    "github.com/gogpu/ui/widget"
)

gogpuApp := gogpu.NewApp(gogpu.DefaultConfig().WithTitle("My App"))
uiApp := app.New(
    app.WithWindowProvider(gogpuApp),
    app.WithPlatformProvider(gogpuApp),
    app.WithEventSource(gogpuApp.EventSource()),
)
uiApp.SetRoot(widget.Text("Hello, GoGPU!"))
desktop.Run(gogpuApp, uiApp)
```

## Dependency Order

```
gputypes (base types)
  ↓
gpucontext (shared interfaces)
  ↓
naga (shader compiler, no GPU deps)
  ↓
wgpu (WebGPU implementation)
  ↓
gogpu (app framework)
  ↓
gg (2D graphics)    g3d (3D rendering)
  ↓
ui (GUI toolkit)
```

When importing multiple packages, ensure compatible versions. All packages follow semantic versioning.

## Build & Test

```bash
# Build any project using gogpu
go build ./...

# Run tests
go test ./...

# Select GPU backend via environment
GOGPU_GRAPHICS_API=vulkan   ./myapp   # Vulkan (Linux/Windows)
GOGPU_GRAPHICS_API=dx12     ./myapp   # DirectX 12 (Windows)
GOGPU_GRAPHICS_API=metal    ./myapp   # Metal (macOS)
GOGPU_GRAPHICS_API=gles     ./myapp   # OpenGL ES
GOGPU_GRAPHICS_API=software ./myapp   # CPU software renderer
```

## Key Conventions

- **Go 1.25+** required
- **Zero CGO** — no C compiler needed, pure `go build`
- **Config builder pattern**: `gogpu.DefaultConfig().WithTitle("...").WithSize(800, 600)`
- **GPU backend selection**: build tags (`-tags rust` for Rust FFI, `GOOS=js GOARCH=wasm` for browser) or env var
- **Error handling**: always check errors, never `_ = err`
- **HiDPI**: `App.Size()` returns logical DIP, `App.PhysicalSize()` for device pixels

## Platform Support

| Platform | Backends |
|----------|----------|
| Windows | Vulkan, DX12, GLES, Software |
| macOS | Metal, Software |
| Linux (X11) | Vulkan, GLES, Software |
| Linux (Wayland) | Vulkan, GLES, Software |
| Browser/WASM | WebGPU |

## Examples

Each repository has an `examples/` directory with working code:

| Repo | Examples | Key demos |
|------|----------|-----------|
| **gogpu** | `examples/triangle/`, `examples/particles/`, `examples/hidpi/` | Window creation, GPU rendering, HiDPI |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gogpu/gogpu](https://github.com/gogpu/gogpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
