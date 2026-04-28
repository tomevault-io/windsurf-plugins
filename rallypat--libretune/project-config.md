---
trigger: always_on
description: LibreTune is a modern, open-source ECU tuning software for Speeduino, EpicEFI, and compatible aftermarket ECUs.
---

# LibreTune - Implementation Guide for AI Agents

## Project Overview
LibreTune is a modern, open-source ECU tuning software for Speeduino, EpicEFI, and compatible aftermarket ECUs.
It's built with Rust core + Tauri desktop app + React frontend.

## Supported ECU Platforms
LibreTune supports multiple ECU platforms, each treated as a distinct project:

| Platform | Description | INI Pattern | Test Coverage |
|----------|-------------|-------------|---------------|
| **Speeduino** | Open-source Arduino-based ECU | `speeduino*.ini` | Full platform test |
| **rusEFI** | Open-source STM32-based ECU | `rusEFI*.ini` (not FOME/epicEFI) | Full platform test |
| **FOME** | Fork of rusEFI with enhanced features | `*FOME*.ini` | All files tested |
| **epicEFI** | rusEFI variant for epicECU boards | `*epicECU*.ini` | Sampled testing (10 files) |
| **MegaSquirt** | MS2/MS3 ECU systems | `MS2*.ini`, `MS3*.ini` | Basic parsing |

**Note**: FOME, epicEFI, and rusEFI are separate projects and should not be conflated in code or documentation.

## Architecture
```
crates/
├── libretune-core/      # Rust library (ECU communication, INI parsing, AutoTune)
└── libretune-app/       # Tauri desktop app
    ├── src/              # React frontend
    ├── src-tauri/        # Tauri backend glue
```

## Tuning Goals
The project aims to provide professional ECU tuning workflow and functionality while:
- Using modern UI patterns (glass-card styling, smooth animations)
- Being open-source and legally distinct
- Improving UX (better keyboard navigation, tooltips, responsive design)

## Key Features Implemented

### 1. Table Editing (2D/3D)
- Location: `crates/libretune-app/src/components/tables/`
- Files:
  - `TableEditor2D.tsx` - Main 2D table editor with toolbar
  - `TableEditor3D.tsx` - 3D visualization (canvas-based)
- Backend: `crates/libretune-core/src/table_ops.rs`
- Features: Set Equal, Increase/Decrease, Scale, Interpolate, Smooth, Re-bin, Copy/Paste, History Trail, Follow Mode

### 2. AutoTune
- Location: `crates/libretune-app/src/components/tuner-ui/AutoTune.tsx`
- Backend: `crates/libretune-core/src/autotune.rs`
- Features: Auto-tuning with recommendations, heat maps, cell locking, filters, authority limits
- **Documentation** (Feb 3, 2026):
  - Created comprehensive usage guide: `docs/src/features/autotune/usage-guide.md`
  - Added step-by-step workflow (Setup → Driving → Review → Apply)
  - Included real-world scenarios (NA, turbo, E85 engines)
  - Troubleshooting section with common issues and solutions
  - Keyboard shortcuts and best practices
  - Multi-location documentation: docs/src and public/manual (kept in sync)

### 3. Dashboard System (TunerStudio-Compatible)
- **NEW**: `crates/libretune-app/src/components/dashboards/TsDashboard.tsx` - Main dashboard component
- **NEW**: `crates/libretune-app/src/components/dashboards/dashTypes.ts` - TypeScript types for TunerStudio format
- **NEW**: `crates/libretune-app/src/components/dashboards/GaugeContextMenu.tsx` - Right-click context menu
- **NEW**: `crates/libretune-app/src/components/gauges/TsGauge.tsx` - Canvas-based gauge renderer
- **NEW**: `crates/libretune-app/src/components/gauges/TsIndicator.tsx` - Boolean indicator renderer
- Backend: `crates/libretune-core/src/dash/{mod.rs,types.rs,parser.rs,writer.rs}`
- Features: Native .ltdash.xml format, TunerStudio .dash import, right-click context menu, designer mode, gauge demo, dashboard selector
- Dashboard Storage: `<app_data>/dashboards/` with auto-generated defaults (Basic, Tuning, Racing)

### 4. Gauge Rendering (TunerStudio-Compatible)
- Location: `crates/libretune-app/src/components/gauges/TsGauge.tsx`
- **Enhanced in Jan 2026**: All gauge types now feature metallic bezels, shadows, gradients, and 3D effects
- Gauge Types Implemented (13 of 13 - ALL COMPLETE):
  - BasicReadout - LCD-style digital numeric display with metallic frame and inset shadows
  - HorizontalBarGauge - Horizontal progress bar with rounded corners and gradient fills
  - VerticalBarGauge - Vertical progress bar with tick marks and 3D gradient effects
  - AnalogGauge - Classic circular dial with metallic bezel, minor ticks, gradient needle, center cap
  - AsymmetricSweepGauge - Curved sweep gauge with glowing tip, warning zones, gradient fills
  - HorizontalLineGauge - Horizontal line indicator with position dot and glow effect
  - VerticalDashedBar - Segmented vertical bar with per-segment zone coloring
  - Histogram - Bar chart distribution visualization centered on current value
  - LineGraph - Time-series line chart with filled gradient area and current value dot
  - RoundGauge - Circular gauge with 270° arc and tick marks
  - RoundDashedGauge - Circular gauge with segmented arc
  - FuelMeter - Specialized fuel level gauge
  - Tachometer - RPM-specific gauge with redline zone

### 5. Professional Default Dashboards
- Location: `crates/libretune-app/src-tauri/src/lib.rs` (create_*_dashboard functions)
- Three professionally designed dashboards:
  - **Basic**: Large analog RPM + digital AFR + vertical CLT/IAT bars + horizontal MAP bar + battery/advance/VE/PW readouts
  - **Racing**: Giant center RPM analog + oil pressure/water temp vertical bars + speed/AFR/boost/fuel digital readouts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RallyPat/LibreTune](https://github.com/RallyPat/LibreTune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
