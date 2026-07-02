---
trigger: always_on
description: AudioResSwitcher-Raycast is a Raycast extension for macOS that enables users to inspect and switch between audio formats on input/output devices with lossless quality control.
---

# CLAUDE.md

## Project Overview

AudioResSwitcher-Raycast is a Raycast extension for macOS that enables users to inspect and switch between audio formats on input/output devices with lossless quality control.

## Development Commands

```bash
# Navigate to extension directory
cd raycast-extension

# Install dependencies
npm install

# Development mode (launches Raycast dev)
npm run dev

# Build for production
npm run build

# Lint code
npm run lint

# Fix lint issues automatically
npm run fix-lint

# Publish to Raycast Store
npm run publish
```

## Development Environment

The project uses a Nix-based development environment:
- **Nix flake** provides reproducible environment with all dependencies
- **direnv** automatically loads the environment when entering the directory
- Run `direnv allow` after first clone to enable automatic environment setup

## Architecture

### Project Structure
```
raycast-extension/
├── src/
│   ├── output-formats.tsx           # Output device UI & logic
│   ├── input-formats.tsx            # Input device UI & logic
│   ├── audio-bitrate-menubar.tsx    # Menubar bitrate monitor
│   └── types.ts                     # TypeScript type definitions
├── swift/
│   ├── Package.swift                # Swift Package Manager manifest
│   └── Sources/
│       └── AudioFormats.swift       # CoreAudio integration (@raycast functions)
└── assets/
    └── command-icon.png             # Extension icon
```

### Component Responsibilities
- **`output-formats.tsx` / `input-formats.tsx`**: React components providing Raycast UI for format selection
- **`audio-bitrate-menubar.tsx`**: MenuBar extra component displaying current audio bitrates
- **`types.ts`**: Shared TypeScript type definitions matching Swift Codable structs
- **`swift/Sources/AudioFormats.swift`**: Swift Package with @raycast-exported functions for CoreAudio operations
  - `getOutputFormats()` / `setOutputFormat()`: Output device format management
  - `getInputFormats()` / `setInputFormat()`: Input device format management
  - `getAudioBitrate()`: Real-time bitrate information retrieval
- **`swift/Package.swift`**: Swift Package Manager manifest with Raycast Swift tools dependencies

### Integration Architecture
1. **Raycast Swift Tools**: Uses official `@raycast` macros for type-safe Swift ↔ TypeScript communication
2. **Build Plugins**: Raycast build plugins auto-generate TypeScript bindings from `@raycast` functions
3. **Import Pattern**: TypeScript imports Swift functions via `import { functionName } from "swift:../swift"`
4. **Type Safety**: Swift `Encodable` structs automatically serialize to TypeScript types
5. **Error Handling**: Swift `throws` errors map to Promise rejections in TypeScript
6. **Build Process**:
   - Raycast CLI (`ray build` / `ray develop`) compiles both TypeScript and Swift
   - Swift Package Manager resolves dependencies and builds Swift code
   - Build plugins generate TypeScript bindings automatically

## Features

### Audio Format Selection Commands
Standard Raycast commands that provide a list interface for switching audio formats:
- **Output Device Formats**: View and switch output device audio formats
- **Input Device Formats**: View and switch input device audio formats
- Both commands display available sample rates, bit depths, and channel configurations
- Quality indicators (crown icon) for lossless formats (192kHz/24-bit+)

### Menubar Bitrate Monitor
Real-time menubar display showing current audio bitrate for both input and output devices:
- **Compact display**: Shows format as `Out: 96k/24 | In: 48k/24` in menubar
- **Quality indicators**: Crown (192k/24+), Star (96k/24+), Circle (standard)
- **Dropdown menu**: Click to see device names, full format details, and quick actions
- **Quick navigation**: Links to Output/Input Format commands for switching
- **Auto-refresh**: Automatically updates when audio device formats change

## Audio Format Management

### Format Detection Strategy
1. **Primary**: Swift CoreAudio API queries for exact supported formats per device
2. **Physical Formats**: Queries `kAudioStreamPropertyAvailablePhysicalFormats` for real hardware capabilities
3. **Format Data**: Sample rate (Hz), bit depth, channel count, format type (Float/Integer)
4. **Deduplication**: Filters duplicate formats to show only unique configurations

### Format Switching
- Swift code directly manipulates CoreAudio device properties via `AudioObjectSetPropertyData`
- Targets `kAudioStreamPropertyPhysicalFormat` for actual hardware format (not virtual/software layer)
- Raycast UI provides format selection with real-time feedback
- Changes take effect immediately without device restart

## Technical Implementation

### Error Handling
```
Swift @raycast Function → Promise → TypeScript → UI Toast
    ├─ Success: Typed result object (e.g., AudioFormatsResult)
    └─ Error: Swift throws → Promise rejection → Error toast
```

### Build System
- **Swift Package Manager**: Manages Swift dependencies and compilation
- **Raycast Swift Tools**: Build plugins generate TypeScript bindings at compile time
- **Type Generation**: `@raycast` macros automatically create TypeScript interfaces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DivitMittal/AudioResSwitcher-Raycast](https://github.com/DivitMittal/AudioResSwitcher-Raycast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
