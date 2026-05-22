---
trigger: always_on
description: - **Use `using` directives** — never write fully qualified type names inline (e.g. `[DefaultValue(...)]` not `[System.ComponentModel.DefaultValue(...)]`).
---

# VL.OCIO Architecture

## C# Code Style Rules
- **Use `using` directives** — never write fully qualified type names inline (e.g. `[DefaultValue(...)]` not `[System.ComponentModel.DefaultValue(...)]`).
- Keep code clean and readable.

OpenColorIO integration for vvvv/Stride with GPU and CPU color transforms, HDR color grading, and a WebSocket-driven web UI for real-time parameter control.

## Project Structure

```
VL.OCIO/
├── src/
│   ├── OCIOSharp/                     # C++ OCIO library + C++/CLI wrapper
│   │   ├── OCIOSharpCLI/
│   │   │   └── OCIOConfig.h           # Main C++/CLI wrapper
│   │   └── OpenColorIO/               # OCIO v2.5 source (submodule)
│   ├── OCIOColorSpacesDynamicEnum.cs  # Dynamic enum definitions (CS, DisplayView, Look, Config)
│   ├── OCIOConfigUtils.cs             # C# utility layer (GPU resources, enum refresh)
│   ├── OCIOConfigService.cs           # Per-app singleton managing config lifecycle
│   ├── OCIOConfigLoader.cs            # ProcessNode: load OCIO config from file
│   ├── OCIOConfigManager.cs           # ProcessNode: switch active config via dropdown
│   ├── OCIOTransformCPU.cs            # CPU colorspace transform node
│   ├── OCIODisplayViewTransformCPU.cs # CPU display/view transform node
│   └── HDR/                           # HDR color grading system
│       ├── ColorSpaceEnums.cs         # HDRColorSpace, DisplayFormat, TonemapOperator, IOColorSpace, GradingSpace, DebugMode
│       ├── ColorCorrectionSettings.cs # Color correction parameters (exposure, contrast, LGG, color wheels)
│       ├── TonemapSettings.cs         # Tonemap parameters (operator, paper white, peak brightness)
│       ├── ProjectSettings.cs         # Combined settings with JSON serialization + presets
│       ├── ColorGradingServer.cs      # WebSocket server ProcessNode for web UI communication
│       └── ColorGradingInstance.cs    # Per-instance color grading state (multi-instance support)
├── shaders/
│   ├── OCIOBase.sdsl                  # Stride shader base (abstract OCIODisplay)
│   ├── OCIOTransform_TextureFX.sdsl   # GPU transform shader host (OCIO LUTs)
│   ├── ColorSpaceConversion.sdsl      # Matrix-based conversions (all HDR color spaces)
│   ├── HDRGrade_TextureFX.sdsl        # GPU color grading (log/linear workflows)
│   └── HDRTonemap_TextureFX.sdsl      # GPU tonemapping + HDR output transforms
├── ui/                                # React/Vite web UI for color grading
│   ├── src/
│   │   ├── App.tsx                    # Main app with color grading controls
│   │   ├── hooks/useWebSocket.ts      # WebSocket connection to ColorGradingServer
│   │   ├── types/settings.ts          # TypeScript settings types (mirrors C#)
│   │   └── components/               # UI components (ColorWheel, Slider, PresetManager, etc.)
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   └── package.json
├── presets/                           # JSON preset files for color grading
│   └── default.json
├── VL.OCIO.vl                         # vvvv node definitions
└── CLAUDE.md                          # This file
```

## Core Components

### 1. C++/CLI Wrapper (`OCIOConfig.h`)

**Purpose:** Bridge between OCIO C++ API and .NET/C#

**Key Methods:**
- `LoadConfig(path)` / `LoadBuiltinConfig(name)` - Load OCIO configs
- `GetColorSpaces()`, `GetDisplays()`, `GetLooks()`, `GetRoles()` - Query config
- `CreateProcessor(...)` - Create transform processor (3 overloads)
  - ColorSpace → ColorSpace
  - Input → Display/View
  - Input → Look → Display/View (GroupTransform via `CreateDisplayViewProcessor`)
- `GetHLSLShader()`, `GetUniforms()`, `GetTextures()`, `Get3DTextures()` - GPU resources
- `ApplyCPUTransform(...)`, `ApplyCPUTransformSeparate(...)`, `ApplyCPUTransformPixel(...)` - CPU pixel processing

**Synthetic Linear Displays:**
- Adds 4 linear displays: Linear Rec.709, Rec.2020, P3-D65, AdobeRGB
- Matrix-only transform (XYZ D65 → target primaries, no gamma)
- Uses OCIO's exact matrices from `ColorMatrixHelpers.cpp`
- Copies ALL views from ALL existing displays (HDR + SDR coverage)

### 2. Config Management System

**OCIOConfigService** (`OCIOConfigService.cs`):
- Per-app singleton via `AppHost.Services.RegisterService()`
- Holds the active `OCIOConfig` and manages switching between configs
- Supports both builtin OCIO configs and file-loaded configs
- `SwitchConfig(name)` - Switch active config, refreshes all enums
- `LoadConfigFromFile(path)` - Load custom config, deduplicates by path, handles name collisions with `#N` suffix
- `EnsureDefaultLoaded()` - Lazy init with "ACES 2.0 CG" default

**OCIOConfigLoader** (`OCIOConfigLoader.cs`):
- ProcessNode that loads an OCIO config from a file path
- Multiple instances can load multiple configs into the dropdown
- Change-detected: only reloads when path changes

**OCIOConfigManager** (`OCIOConfigManager.cs`):
- ProcessNode to switch active config via `OCIOConfigEnum` dropdown
- Outputs formatted list of all available configs with sources

**OCIOConfigUtils** (`OCIOConfigUtils.cs`):
- Static utility layer, holds `ActiveConfig` reference
- `RefreshEnumsFrom(config)` - Rebuilds all dynamic enums from config (3 triggers total)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tebjan/VL.OCIO](https://github.com/tebjan/VL.OCIO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
