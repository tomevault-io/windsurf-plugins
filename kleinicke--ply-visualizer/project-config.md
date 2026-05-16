---
trigger: always_on
description: manages:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Development Commands

### Build & Development

```bash
npm run compile          # Build the extension using webpack
npm run watch           # Watch mode for development
npm run vscode:prepublish  # Production build before publishing
npm run clean           # Clean output directory
npm run compile:all     # Compile both extension and tests
```

### Testing & Linting

```bash
npm run test            # Run unit tests (Mocha-based)
npm run lint            # Run ESLint on TypeScript files
npm run pretest         # Compile and lint before testing
npm run test:ui         # Run UI tests (VS Code Extension Tester)
npm run test:all        # Run both unit and UI tests
npm run test:stl        # Quick STL functionality validation and checklist
npm run test:coverage   # Run tests with coverage analysis
npm run coverage        # Generate coverage report
```

### Code Quality & Formatting

```bash
npm run format          # Format code with Prettier
npm run format:check    # Check code formatting
npx lint-staged         # Run linting on staged files (via git hooks)
```

### Extension Testing

- Use **F5** in VS Code to launch Extension Development Host
- Test files organized by format in `testfiles/` subdirectories:
  - `testfiles/stl/` - STL triangle mesh files (ASCII/binary)
  - `testfiles/ply/` - PLY point cloud files and XYZ coordinates
  - `testfiles/np/` - NPY/NPZ depth/disparity arrays
  - `testfiles/tif/` - TIF/TIFF depth images
  - `testfiles/png/` - PNG depth images
  - `testfiles/pfm/` - PFM depth files
  - `testfiles/json/` - JSON pose/keypoint files
  - `testfiles/obj/` - OBJ wireframe files (if present)
  - Root directory - Large test PLY files for performance testing

### Development Workflow

**For Core Functionality Changes** (parsers, visualization, controls):

1. Work in `website/src/` directory - this is the shared codebase
2. Test standalone website: `cd website && npm run dev` (if available)
3. Test VS Code extension: Press **F5** to launch Extension Development Host
4. Changes in `website/src/` automatically affect both targets

**For VS Code-Specific Features** (commands, menus, file associations):

1. Work in `src/` directory for extension host integration
2. Use `npm run watch` for automatic rebuilding during development
3. Test with **F5** Extension Development Host

## Architecture Overview

This project implements a **dual-target architecture** supporting both a
standalone website and a VS Code extension:

1. **Standalone Website** (`website/` folder) - Can run independently at
   https://f-kleinicke.de
2. **VS Code Extension** - Integrates the same core functionality into VS Code

**Key Architectural Principle**: The `website/src/` directory contains all core
visualization functionality (parsers, renderers, controls, depth processing)
that is shared between both targets. The `src/` directory contains only VS
Code-specific integration code.

### Project Structure

```
├── src/                     # VS Code extension-specific files
│   ├── extension.ts         # Extension activation & VS Code API integration
│   ├── pointCloudEditorProvider.ts  # Custom editor registration
│   └── *Parser.ts          # Lightweight parser wrappers for extension host
├── website/                 # Shared core functionality + standalone website
│   ├── src/                # Core visualization engine (shared code)
│   │   ├── main.ts         # Main 3D visualization engine (~15,576 lines - TOO BIG!)
│   │   ├── fileHandler.ts  # Shared file handling logic (USE THIS!)
│   │   ├── controls.ts     # Camera control systems (USE THIS!)
│   │   ├── interfaces.ts   # Shared type definitions
│   │   ├── parsers/        # Complete format parsers (ADD NEW PARSERS HERE!)
│   │   │   ├── plyParser.ts
│   │   │   ├── objParser.ts
│   │   │   ├── stlParser.ts
│   │   │   └── ...
│   │   ├── depth/          # Depth-to-pointcloud processing (ADD DEPTH FEATURES HERE!)
│   │   │   ├── DepthRegistry.ts
│   │   │   ├── DepthProjector.ts
│   │   │   ├── readers/    # Format-specific readers (ADD NEW READERS HERE!)
│   │   │   └── ...
│   │   ├── themes/         # UI themes and styling (ADD THEME FEATURES HERE!)
│   │   │   ├── darkModern.ts
│   │   │   └── ...
│   │   ├── ui/             # UI generation modules (CREATE IF NEEDED)
│   │   │   ├── dialogs.ts  # Dialog HTML generators (proposed)
│   │   │   └── ...
│   │   └── utils/          # Utility modules (CREATE IF NEEDED)
│   │       ├── math.ts     # Math/geometry helpers (proposed)
│   │       └── ...
│   ├── index.html          # Standalone website entry point (SINGLE SOURCE OF TRUTH!)
│   └── webpack.config.js   # Website-specific build configuration
├── media/                  # Shared static assets (CSS, external libraries)
└── testfiles/             # Test data organized by format type
```

This architecture enables:

- **Code Reuse**: Core functionality written once, used in both contexts
- **Independent Development**: Website can be developed and tested standalone
- **VS Code Integration**: Extension provides native VS Code experience with
  same features

### Core Components

**Extension Host (Node.js)**

- `src/extension.ts` - Extension activation and command registration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kleinicke/ply-visualizer](https://github.com/kleinicke/ply-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
