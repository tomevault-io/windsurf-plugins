---
trigger: always_on
description: **See also:** [ARCHITECTURE.md](ARCHITECTURE.md) (structure & conventions), [GUARDRAILS.md](GUARDRAILS.md) (must / must-not), [.cursorrules](.cursorrules).
---

# XLKit - AI Agent Development Guide

**See also:** [ARCHITECTURE.md](ARCHITECTURE.md) (structure & conventions), [GUARDRAILS.md](GUARDRAILS.md) (must / must-not), [.cursorrules](.cursorrules).

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
  - [Module Structure](#module-structure)
  - [Dependencies](#dependencies)
  - [Test Runner](#test-runner)
- [Development Standards](#development-standards)
  - [Code Quality](#code-quality)
  - [Security Requirements](#security-requirements)
  - [Security Features](#security-features)
  - [Testing Strategy](#testing-strategy)
  - [Documentation](#documentation)
- [API Reference](#api-reference)
  - [Core Types](#core-types)
  - [Sheet Operations](#sheet-operations)
  - [Workbook Operations](#workbook-operations)
  - [CSV/TSV Operations](#csvtsv-operations)
  - [Image Operations](#image-operations)
- [Recent Improvements](#recent-improvements)
  - [Comprehensive Demo and Sheet Password Utilities](#comprehensive-demo-and-sheet-password-utilities-2026-06-03-116)
  - [Swift Testing Migration and CI Updates](#swift-testing-migration-and-ci-updates-2026-07-18-unreleased-117)
  - [Sheet Visibility and Protection](#sheet-visibility-and-protection-2026-05-30-pr-23)
  - [iOS Compatibility Fix](#ios-compatibility-fix-2025-07-14)
  - [Perfect Aspect Ratio Preservation](#perfect-aspect-ratio-preservation-2025-07-07)
  - [Scaling API Investigation and Fixes](#scaling-api-investigation-and-fixes-2025-07-12)
- [Implementation Details](#implementation-details)
  - [XLSX Generation](#xlsx-generation)
  - [Image Embedding Scaling API](#image-embedding-scaling-api)
  - [Image Embedding](#image-embedding)
  - [Error Handling](#error-handling)
- [Performance & Optimization](#performance--optimization)
- [Maintenance & Updates](#maintenance--updates)

---

## Project Overview

XLKit is a modern Swift library for creating and manipulating Excel (.xlsx) files on macOS and iOS. Built with Swift 6.0, targeting macOS 12+ and iOS 15+, using modular Swift Package Manager architecture. iOS support is available and tested in CI/CD, with platform-specific code handling for iOS compatibility.

## Architecture

### Module Structure

The library is organized into five SPM modules:

- **XLKitCore**: Core types, data structures, and utilities
- **XLKitFormatters**: CSV/TSV import/export functionality  
- **XLKitImages**: Image processing and embedding utilities
- **XLKitXLSX**: XLSX file generation engine
- **XLKit**: Main API that re-exports all submodules

### Directory and File Structure

```
XLKit/
├── AGENT.MD                     # AI agent development guide
├── ARCHITECTURE.md              # Module stack, save pipeline, conventions
├── GUARDRAILS.md                # Must / must-not for contributors and agents
├── .cursorrules                 # Cursor rules for AI agents
├── CHANGELOG.md                 # Version history and changes
├── LICENSE                      # MIT license
├── Package.swift                # Swift Package Manager configuration
├── Package.resolved             # Locked dependency versions
├── README.md                    # Main documentation
├── SECURITY.md                  # Security policy
├── .gitignore                   # Git ignore patterns
├── .swift-format                # Swift formatting configuration
├── Assets/                      # Project assets
│   └── XLKit_Icon.png          # Project icon
├── Sources/                     # Source code modules
│   ├── XLKit/                  # Main API module
│   │   ├── XLKit.swift         # Main API exports
│   │   ├── Sheet+API.swift     # Sheet operations API
│   │   └── Workbook+API.swift  # Workbook operations API
│   ├── XLKitCore/              # Core types and utilities
│   │   ├── CoreTypes.swift     # Core data structures (1253 lines)
│   │   └── SecurityManager.swift # Security features (282 lines)
│   ├── XLKitFormatters/        # CSV/TSV functionality
│   │   └── CSVUtils.swift      # CSV import/export utilities (294 lines, uses swift-textfile)
│   ├── XLKitImages/            # Image processing
│   │   ├── ImageUtils.swift    # Image utilities (155 lines)
│   │   └── ImageSizingUtils.swift # Image sizing logic (191 lines)
│   ├── XLKitXLSX/              # XLSX generation engine
│   │   └── XLSXEngine.swift    # XLSX file generation (897 lines)
│   └── XLKitTestRunner/        # Test runner executable
│       ├── main.swift          # Command-line interface (91 lines)
│       ├── ExcelGenerators.swift # Excel generation tests (590 lines)
│       ├── ImageEmbedGenerators.swift # Image embedding tests (228 lines)
│       ├── SheetPasswordUtilities.swift # sheet-password CLI output
│       ├── ComprehensiveDemoProtection.swift # Demo password/salts for comprehensive CLI
│       ├── README.md           # Test runner documentation
│       └── Templates/          # Test templates
│           └── TestGeneratorTemplate.swift # Template for new tests (224 lines)
├── Documentation/              # User manual (Documentation/Manual/)
├── Tests/                      # Unit tests
│   ├── README.md               # Test suite index (80 tests)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAcharya/XLKit](https://github.com/TheAcharya/XLKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
