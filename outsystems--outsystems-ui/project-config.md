---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

OutSystems UI is a UI component library providing TypeScript behaviors and CSS styles for 70+ patterns used in OutSystems Reactive Web and Native Mobile applications. The compiled library runs in end users' browsers and is consumed by applications built in OutSystems Service Studio.

**Key references:**

- See [ARCHITECTURE.md](./ARCHITECTURE.md) for system design, architectural tenets (Provider Pattern Isolation, Two-Tier Namespace Separation, Pattern Registry, Platform-Specific Compilation Guards, Factory Pattern), and external integrations table
- See [CONTRIBUTING.md](./CONTRIBUTING.md) for development setup, workflow, code standards, PR requirements, and testing procedures

## Command Quick Reference

```bash
# Setup and development
npm run setup              # Install dependencies + start dev server
npm run dev                # Start dev server for all platforms (http://localhost:3000)
npm run dev -- --target O11    # Start dev server for O11 platform only
npm run dev -- --target ODC    # Start dev server for ODC platform only

# Building and quality checks
npm run build              # Production build (all platforms) + lint + format
npm run lint               # Check ESLint rules
npm run lintfix            # Auto-fix ESLint issues
npm run prettier           # Format all code

# Documentation
npm run docs               # Generate TypeDoc documentation
```

**Build output location:** `dist/` directory contains compiled JavaScript and CSS bundles per platform.

## Directory Structure

```
src/
├── scripts/
│   ├── OSFramework/           # Internal framework (not invoked directly)
│   │   └── OSUI/
│   │       ├── Behaviors/      # Pattern behavior classes
│   │       ├── Event/         # Event management (DOM, Gesture, Provider)
│   │       ├── Helper/        # Utilities (Dom, Dates, Device, Sanitize)
│   │       ├── Interface/     # Generic interfaces
│   │       ├── Pattern/       # Pattern implementations (Accordion, Carousel, etc.)
│   │       ├── Constants.ts   # Framework constants
│   │       ├── ErrorCodes.ts  # Internal error codes
│   │       └── GlobalEnum.ts  # Framework enumerations
│   │
│   ├── OutSystems/            # Public APIs (consumed by OutSystems apps)
│   │   └── OSUI/
│   │       ├── Patterns/      # Pattern APIs (AccordionAPI.ts, CarouselAPI.ts, etc.)
│   │       ├── Utils/         # Public utility APIs
│   │       └── ErrorCodes.ts  # Public error codes
│   │
│   ├── Providers/             # Third-party library integrations
│   │   └── OSUI/
│   │       ├── Carousel/            # Splide provider for Carousel
│   │       ├── Datepicker/          # Flatpickr for DatePicker
│   │       ├── Dropdown/            # VirtualSelect for Dropdown
│   │       ├── Monthpicker/         # Flatpickr for MonthPicker
│   │       ├── RangeSlider/         # NoUiSlider for RangeSlider
│   │       ├── Timepicker/          # Flatpickr for TimePicker
│   │       ├── SharedProviderResources/  # Shared Flatpickr resources
│   │       └── Utils/               # FloatingUI for positioning
│   │
│   └── osui.ts                # Deprecated API (redirects to OutSystems.OSUI.*)
│
├── scss/                      # Styles (non-pattern specific)
│   ├── 00-abstract/           # Variables, mixins, functions
│   ├── 01-foundations/        # Base styles, typography, colors
│   ├── 02-layout/             # Grid, layout utilities
│   ├── 03-widgets/            # Widget-specific styles
│   ├── 04-patterns/           # Pattern styles by category
│   └── 05-useful/             # Utility classes
│
gulp/                          # Build system configuration
├── Tasks/                     # Gulp tasks (TsTranspile, ScssTranspile, etc.)
├── ProjectSpecs/              # Build specifications
└── Template/                  # index.html template for dev server
```

See `src/README.md` for detailed directory structure documentation.

## OutSystems Domain Concepts

**Pattern:** A reusable UI component (e.g., Accordion, Carousel, DatePicker). Each pattern has:

- Internal implementation in `OSFramework/OSUI/Pattern/<PatternName>/`
- Public API in `OutSystems/OSUI/Patterns/<PatternName>API.ts`
- Configuration class ending in `Config.ts`
- Optional provider integration for complex patterns

**Provider:** Third-party library wrapped by the framework (Splide, Flatpickr, NoUiSlider, VirtualSelect, FloatingUI). Providers are isolated from public APIs using the Provider Pattern (see ARCHITECTURE.md T1. Provider Pattern Isolation).

**Platform Targets:**

- **O11** - OutSystems 11 (traditional platform)
- **ODC** - OutSystems Developer Cloud (cloud-native platform)

Build system compiles separate bundles for each platform with platform-specific code exclusions and placeholder replacements.

**Forge:** OutSystems component marketplace where this library is distributed.

**Service Studio:** OutSystems IDE where developers drag/drop UI patterns from this library.

## Code Patterns and Conventions

### TypeScript Naming (enforced by ESLint)

```typescript
// Classes: StrictPascalCase
class AccordionConfig { }

// Interfaces: IPascalCase or UPPER_CASE with I prefix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OutSystems/outsystems-ui](https://github.com/OutSystems/outsystems-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
