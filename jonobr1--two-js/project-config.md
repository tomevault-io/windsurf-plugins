---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Two.js is a renderer-agnostic 2D drawing API for modern browsers. It provides a unified interface for creating graphics across multiple rendering contexts: WebGL, Canvas2D, and SVG.

## Core Architecture

### Main Entry Point
- `src/two.js` - Main Two.js class and entry point that imports all modules
- The Two class extends Events and provides factory methods for creating shapes

### Rendering System
- **Multi-renderer architecture**: Canvas, SVG, and WebGL renderers in `src/renderers/`
- **Scene graph**: Hierarchical structure using Groups and Elements
- **Automatic renderer selection**: Based on domElement type or explicit type specification

### Core Classes
- `Element` - Base class for all drawable objects
- `Shape` - Extended Element with transformation and styling
- `Group` - Container for organizing and transforming multiple objects
- `Path` - Complex shapes defined by anchor points and curves
- `Vector` - 2D vector mathematics
- `Matrix` - 2D transformation matrices
- `Anchor` - Control points for paths with B�zier curve handles

### Shape Library
Located in `src/shapes/`:
- Basic shapes: Rectangle, Circle, Ellipse, Line, Star, Polygon
- Complex shapes: ArcSegment, RoundedRectangle, Points
- All shapes inherit from Path or Shape classes

### Effects System
Located in `src/effects/`:
- Gradients: LinearGradient, RadialGradient with Stop objects
- Images: Texture, Sprite, ImageSequence for bitmap rendering
- All effects can be applied as fill or stroke to shapes

## Build System

### Commands
- `npm run build` - Build all versions (UMD, ESM, minified) using esbuild
- `npm run dev` - Development server with esbuild on port 8080
- `npm run lint` - ESLint with auto-fix
- `npm run docs:generate` - Generate documentation from JSDoc comments
- `npm run docs:dev` - Local documentation server with Vuepress
- `npm run docs:build` - Build static documentation site

### Build Configuration
- Build script: `utils/build.js`
- Uses esbuild for fast bundling and minification
- Outputs: `build/two.js` (UMD), `build/two.module.js` (ESM), `build/two.min.js` (minified)
- Includes license header and module.exports compatibility

## Development Patterns

### Factory Methods
The Two class provides factory methods for creating and adding objects to the scene:
- `makeRectangle()`, `makeCircle()`, `makeText()`, etc.
- All factory methods automatically add objects to the scene
- Return the created object for further manipulation

### Event System
- All objects inherit from Events class
- Common events: update, render, resize, play, pause
- Use `bind()`, `unbind()`, `trigger()` for event handling

### Coordinate System
- Origin (0,0) at top-left by default
- Positive Y axis points down
- Transformations applied via translation, rotation, scale properties

### Memory Management
- Use `release()` method to unbind events and free memory
- Automatically handles nested objects, vertices, and effects
- Important for preventing memory leaks in long-running applications

## Testing

### Test Structure
- Tests located in `tests/` directory
- Test suites in `tests/suite/` organized by functionality
- HTML test runners: `tests/index.html`, `tests/noWebGL.html`
- TypeScript compilation tests in `tests/typescript/` with `index.ts` that imports and uses Two.js API

### Running Tests
- Manual browser testing via HTML files: `tests/index.html` and `tests/noWebGL.html`
- TypeScript compilation testing: `npx tsc --noEmit --skipLibCheck tests/typescript/index.ts` to verify types work correctly

## Key Files to Understand

- `src/two.js` - Main class with factory methods and core logic
- `src/constants.js` - Global constants, types, and configuration
- `src/utils/interpret-svg.js` - SVG parsing and import functionality
- `utils/build.js` - Build system configuration
- `src/**/*.d.ts` - TypeScript definitions collocated with source files (e.g., `src/vector.d.ts` alongside `src/vector.js`)
- `src/two.d.ts` - Main TypeScript entry point that aggregates all type exports

## Dependencies

Production: None (library designed to be dependency-free)
Development: esbuild, ESLint, TypeScript, Vuepress for documentation

## Browser Compatibility

Designed for modern browsers with ES6+ support. Uses feature detection for renderer capabilities.

## Development Workflow

- Always run `npm run build && npm run lint` before committing
- Test changes in `tests/index.html` for visual verification
- Use `npm run dev` for development server on port 8080
- Check TypeScript types with `npm run types`
- Test across all three renderers (Canvas, SVG, WebGL) for compatibility

## Code Style and Conventions

- Use ES6+ features consistently
- Prefer `const` over `let` where possible
- Factory methods should always return the created object
- All classes should extend appropriate base classes (Element, Shape, etc.)
- Use JSDoc comments for public API methods
- Use 2-space indentation for JavaScript files
- Place new components in appropriate src/ subdirectories
- Application runs lots of functions on requestAnimationFrame (or per animation frame) so:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonobr1/two.js](https://github.com/jonobr1/two.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
