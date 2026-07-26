---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SVGIconImageList is a comprehensive Delphi component library for rendering SVG images in both VCL and FMX applications. The library provides four main components to simplify the use of SVG icons with features like resizing, fixed colors, grayscale conversion, and opacity control.

**Supported Platforms:**
- VCL: Windows (Delphi XE3 to Delphi 13)
- FMX: Cross-platform including Windows, macOS, iOS, Android, Linux

## Build System

### Package Structure

Packages are organized by Delphi version in `Packages/` directory:
- `D10_4/` - Delphi 10.4 Sydney
- `D11/` - Delphi 11 Alexandria
- `D12/` - Delphi 12 Athens
- `D13/` - Delphi 13 Florence
- `DXE3/`, `DXE6/`, `DXE7/`, `DXE8/` - Legacy Delphi versions
- And various D10.x versions

### Main Packages

For each Delphi version, there are typically 4-6 packages:

**VCL Packages:**
1. `SVGIconImageList.dpk` - Runtime VCL components package
2. `dclSVGIconImageList.dpk` - Design-time VCL components package
3. `SVGIconImageListRestClient.dpk` - REST API client for downloading icons from iconify.design
4. `SVGImage32Package.dpk` - Image32 SVG rendering engine package
5. `SVGMagicPackage.dpk` - SVGMagic SVG rendering engine package (supports animated SVG)
6. `dclSVGMagicPackage.dpk` - Design-time SVGMagic package

**FMX Packages:**
1. `SVGIconImageListFMX.dpk` - Runtime FMX components package
2. `dclSVGIconImageListFMX.dpk` - Design-time FMX components package

### Building Packages

The project includes an automatic installer (`Setup/`) that detects Delphi versions, builds, and installs packages. For manual building:

1. Open the appropriate package for your Delphi version in `Packages/[version]/`
2. Build in this order:
   - `SVGImage32Package.dpk` (if using Image32 engine)
   - `SVGMagicPackage.dpk` (if using SVGMagic engine)
   - `SVGIconImageListRestClient.dpk`
   - `SVGIconImageList.dpk` or `SVGIconImageListFMX.dpk`
   - Design-time packages last: `dclSVGIconImageList.dpk`, `dclSVGMagicPackage.dpk`, or `dclSVGIconImageListFMX.dpk`

## SVG Rendering Engines

The library supports four different SVG rendering engines, configured via `Source/SVGIconImageList.inc`:

### For VCL Applications

**Image32 (Default):** `{$DEFINE Image32_SVGEngine}`
- Native Delphi implementation by Angus Johnson
- Included in `Image32/` directory
- Supports blur effects
- Good performance across different icon types

**SVGMagic:** `{$DEFINE SVGMagic_SVGEngine}`
- Advanced SVG implementation by Ursa Minor Ltd.
- Included in `SVGMagic/` directory
- **Unique feature: Supports animated SVG files**
- Comprehensive SVG 1.1 support with advanced rendering capabilities
- Excellent rendering quality for complex SVG files

**Skia4Delphi:** `{$DEFINE Skia_SVGEngine}`
- Uses Google's Skia Graphics Library wrapper
- Requires separate Skia4Delphi installation
- Supports blur effects
- Best performance with simple icons

**Direct2D (Windows only):** `{$DEFINE PreferNativeSvgSupport}`
- Native Windows implementation wrapper
- Requires Windows 10 Creators Update or later
- Does NOT support blur effects or text elements
- Can enable GPU acceleration with `{$DEFINE GPUSupport}`

### For FMX Applications

**Image32 (Default):** `{$DEFINE FMX_Image32_SVGEngine}`
- Cross-platform native Delphi implementation
- Supports all FMX platforms including mobile

**Skia4Delphi:** `{$DEFINE FMX_Skia_SVGEngine}`
- Cross-platform Skia Graphics Library wrapper

**Important:** Only ONE engine can be active at a time per platform (VCL/FMX). The `.inc` file enforces this with compiler directives.

## Core Architecture

### Component Hierarchy

```
TSVGIconImageListBase (base class)
├── TSVGIconImageList - VCL ImageList with embedded SVG collection
├── TSVGIconVirtualImageList - VCL Virtual ImageList (from D10.3+, inherits from TVirtualImageList)
└── TSVGIconImageListFMX - FMX ImageList

TSVGIconImageCollection - Centralized collection of SVG images (VCL only, from D10.4+ inherits from TCustomImageCollection)

TSVGIconImage - VCL/FMX Image component for displaying single SVG icons
```

### Key Source Files

**Core Components (VCL):**
- `SVGIconImageList.pas` - Main ImageList component
- `SVGIconImageCollection.pas` - Image collection component
- `SVGIconVirtualImageList.pas` - Virtual ImageList component
- `SVGIconImage.pas` - Single image display component
- `SVGIconImageListBase.pas` - Base class with common functionality

**Core Components (FMX):**
- `FMX.SVGIconImageList.pas` - FMX ImageList component
- `FMX.SVGIconImage.pas` - FMX Image component
- `FMX.SVGIconsUtils.pas` - FMX utilities

**Infrastructure:**
- `SVGInterfaces.pas` - Core interfaces (ISVG, ISVGFactory) for rendering abstraction
- `SVGIconItems.pas` - SVG icon item and collection classes
- `SVGIconUtils.pas` - Utility functions
- `SVGMessaging.pas` - DPI change messaging support

**Engine Implementations:**
- `Image32SVGFactory.pas` - Image32 engine factory
- `SVGMagicFactory.pas` - SVGMagic engine factory
- `SkiaSVGFactory.pas` - Skia4Delphi engine factory
- `D2DSVGFactory.pas` - Direct2D engine factory
- `FMX.Image32SVG.pas` - FMX Image32 implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EtheaDev/SVGIconImageList](https://github.com/EtheaDev/SVGIconImageList) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
