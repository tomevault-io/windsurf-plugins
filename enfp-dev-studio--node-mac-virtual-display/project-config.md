---
trigger: always_on
description: **node-mac-virtual-display** is a Native Node.js addon for macOS that enables creation and management of virtual displays. The library interfaces with macOS CoreGraphics and CoreDisplay APIs to provide programmatic control over virtual displays.
---

# CLAUDE.md - AI Assistant Guide for node-mac-virtual-display

## Project Overview

**node-mac-virtual-display** is a Native Node.js addon for macOS that enables creation and management of virtual displays. The library interfaces with macOS CoreGraphics and CoreDisplay APIs to provide programmatic control over virtual displays.

**Key Information:**
- **Language:** Objective-C++ (`.mm`), JavaScript, TypeScript definitions
- **Platform:** macOS 10.14+ only
- **Node.js:** v12+
- **License:** MIT
- **Version:** 1.0.9
- **Primary Use Case:** Used in [Tab Display](https://tab-display.enfpdev.com) for tablet-as-monitor functionality

## Codebase Structure

```
node-mac-virtual-display/
├── src/
│   └── virtual_display.mm       # Main C++ native addon implementation
├── test/
│   └── module.spec.js           # Mocha test suite
├── .github/
│   ├── workflows/
│   │   └── release-package.yml  # CI/CD for package publishing
│   └── FUNDING.yml              # Funding configuration
├── index.js                     # JavaScript wrapper/entry point
├── index.d.ts                   # TypeScript type definitions
├── binding.gyp                  # Node-gyp build configuration
├── package.json                 # NPM package configuration
├── README.md                    # User-facing documentation
└── LICENSE                      # MIT License
```

## Architecture & Design

### Core Components

1. **Native C++ Layer** (`src/virtual_display.mm`)
   - Implements `VDisplay` class using N-API (Node Addon API)
   - Interfaces with private macOS frameworks:
     - `CGVirtualDisplay` - Main virtual display controller
     - `CGVirtualDisplayDescriptor` - Display hardware descriptor
     - `CGVirtualDisplaySettings` - Display mode settings
     - `CGVirtualDisplayMode` - Resolution/refresh rate configuration

2. **JavaScript Wrapper** (`index.js`)
   - Exports `VirtualDisplay` constructor
   - Provides clean API over native addon
   - Three main methods:
     - `createVirtualDisplay()` - Create custom display
     - `cloneVirtualDisplay()` - Clone main display
     - `destroyVirtualDisplay()` - Remove virtual display

3. **TypeScript Definitions** (`index.d.ts`)
   - Type-safe interface definitions
   - Exports `DisplayInfo` type

### Key Design Patterns

- **Object-Oriented Wrapper:** JavaScript class wraps native addon instance
- **Resource Management:** Manual memory management in C++ with explicit cleanup
- **Configuration Objects:** Options passed as JavaScript objects with destructuring
- **Mirror Mode Handling:** Post-processing logic to prevent unintended main display changes

## Critical Implementation Details

### Display Creation Logic

When creating a virtual display, the code performs critical post-processing (lines 149-192 in `virtual_display.mm`):

1. **Main Display Restoration:** If virtual display becomes main display unintentionally, restore original
2. **Mirror Prevention:** Prevent primary display from mirroring virtual display
3. **Mirror Mode Configuration:** Apply user's mirror preference (extend vs mirror mode)

**IMPORTANT:** This post-processing logic is essential and should NOT be removed or modified without deep understanding of macOS display behavior.

### Parameter Constraints

- **Refresh Rate:** Clamped to 30-60 Hz range
- **PPI:** Clamped to 72-300 range
- **HiDPI Mode:** When enabled, creates both full-res and half-res modes

### Memory Management

The native addon uses manual memory management:
- Objects allocated with `[[Class alloc] init]`
- Must be released with `[object release]` in `DestroyVirtualDisplay`
- Potential memory leak if display not properly destroyed

## Development Workflow

### Build System

**Technology:** node-gyp (native addon build tool)

**Commands:**
```bash
npm run build      # Rebuild native addon (node-gyp rebuild)
npm run clean      # Clean build artifacts (node-gyp clean)
```

**Build Configuration** (`binding.gyp`):
- Target: `virtual_display.node`
- Compiler: Clang with C++14 standard
- macOS Deployment Target: 10.14
- Framework Dependencies: StoreKit
- Compiler Flags: `-std=c++14 -stdlib=libc++`
- N-API Exception Mode: `NAPI_DISABLE_CPP_EXCEPTIONS`

### Testing

**Framework:** Mocha + Chai

**Command:**
```bash
npm test           # Run test suite
```

**Test Characteristics:**
- Located in `test/module.spec.js`
- Tests create display and wait 10 minutes before cleanup (600000ms timeout)
- Only basic smoke test - verifies module doesn't throw on initialization
- Clone test commented out by default

**IMPORTANT:** Tests create real virtual displays - must run on macOS with proper permissions.

### Code Quality & Formatting

**Linting:**
```bash
npm run lint       # Check formatting (dry-run)
npm run format     # Apply formatting
```

**Tools:**
- **JavaScript:** Prettier (`.js` files)
- **Objective-C++:** clang-format (`.mm` files)

**Git Hooks:**
- Husky configured for pre-commit hooks
- lint-staged runs formatters automatically:
  - `*.js` → prettier
  - `*.mm` → clang-format

### CI/CD Pipeline

**Workflow:** `.github/workflows/release-package.yml`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enfp-dev-studio/node-mac-virtual-display](https://github.com/enfp-dev-studio/node-mac-virtual-display) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
