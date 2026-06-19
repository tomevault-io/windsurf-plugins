---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FlashForgeWebUI is a standalone web-based interface for controlling and monitoring FlashForge 3D printers. It provides a lightweight deployment option for low-spec devices like Raspberry Pi, without Electron dependencies.

**Current Status**: Production-ready. Core functionality tested and working including multi-printer support, Spoolman integration, Discord webhook notifications, go2rtc-based camera streaming, and cross-platform binary distribution.

## Build & Development Commands

### Development
```bash
npm run dev              # Build and watch with hot reload (concurrent backend + webui + server)
npm run build            # Full production build (backend + webui)
npm run build:watch      # Watch backend and frontend builds without starting the server
npm run start            # Run the built application
npm run start:dev        # Run with nodemon (watches for changes)
```

### Build Components
```bash
npm run build:backend           # Bundle backend with esbuild (scripts/build-backend.ts)
npm run build:backend:watch     # Watch backend files
npm run build:webui             # Compile frontend TS + copy static assets
npm run build:webui:watch       # Watch frontend files
npm run build:webui:copy        # Copy HTML/CSS and vendor libraries to dist
```

### Platform-Specific Builds
```bash
npm run build:linux             # Linux x64 executable (using pkg)
npm run build:linux-arm         # Linux ARM64 executable
npm run build:linux-armv7       # Linux ARMv7 executable
npm run build:win               # Windows x64 executable
npm run build:mac               # macOS x64 executable
npm run build:mac-arm           # macOS ARM64 executable
npm run build:all               # Build for all platforms
npm run build:wrapper           # Run the platform build wrapper directly
npm run build:win:wrapped       # Windows x64 build via wrapper
npm run build:linux:wrapped     # Linux x64 build via wrapper
npm run build:linux-arm:wrapped # Linux ARM64 build via wrapper
npm run build:linux-armv7:wrapped # Linux ARMv7 build via wrapper
npm run build:mac:wrapped       # macOS x64 build via wrapper
npm run build:mac-arm:wrapped   # macOS ARM64 build via wrapper
npm run build:all:wrapped       # All wrapped platform builds
```

### Code Quality
```bash
npm run lint              # Run Biome lint checks
npm run lint:fix          # Auto-fix Biome lint issues
npm run format            # Preview Biome formatting changes
npm run format:fix        # Apply Biome formatting changes
npm run check             # Run Biome check (lint + format combined)
npm run check:fix         # Auto-fix Biome check issues
npm run type-check        # TypeScript type checking (app + e2e)
npm run type-check:app    # Type check main application only
npm run type-check:e2e    # Type check e2e tests only (tsconfig.e2e.json)
npm run docs:check        # Validate @fileoverview coverage in source files
npm run docs:check:debug  # Debug fileoverview validation output
npm run clean             # Remove dist directory
npm run download:go2rtc   # Manually download go2rtc binary
```

### Testing
```bash
# Jest app tests
npm test                            # Run all Jest tests
npm run test:watch                  # Jest watch mode
npm run test:coverage               # Jest with coverage
npm run test:verbose                # Jest verbose output

# Playwright fixture E2E (fast, stub server)
npm run test:e2e:install            # Install Chromium for Playwright
npm run test:e2e                    # Run all fixture E2E specs
npm run test:e2e:smoke              # Smoke tests only
npm run test:e2e:auth               # Auth tests only

# Playwright emulator E2E (full server + emulator, workers=1)
npm run test:e2e:emulator           # Run all emulator E2E specs
npm run test:e2e:emulator:direct    # Direct connection spec
npm run test:e2e:emulator:discovery # Discovery spec
npm run test:e2e:emulator:multi    # Multi-printer spec

# Combined
npm run test:e2e:all                # All Playwright suites (fixture + emulator)
npm run test:all                    # Everything (Jest + all Playwright)

# Passthrough: append extra args after --
# npm run test:e2e -- --grep "login"
# npm run test:e2e:emulator:direct -- --grep "connect"
# npm test -- --testPathPattern=Config
```

## Runtime Modes

The application supports multiple startup modes via CLI arguments:

```bash
# Connect to last used printer
node dist/index.js --last-used

# Connect to all saved printers
node dist/index.js --all-saved-printers

# Connect to specific printers
node dist/index.js --printers="192.168.1.100:new:12345678,192.168.1.101:legacy"

# Start without printer connections (WebUI only)
node dist/index.js --no-printers

# Override WebUI settings
node dist/index.js --last-used --webui-port=3001 --webui-password=mypassword
```

Printer spec format: `IP:TYPE:CHECKCODE` where TYPE is `new` or `legacy`.

## Architecture

### Core Architecture Pattern

The system is built on a **multi-context singleton architecture**:

1. **Singleton Managers**: Global coordinators for major subsystems.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Parallel-7/FlashForgeWebUI](https://github.com/Parallel-7/FlashForgeWebUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
