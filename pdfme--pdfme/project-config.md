---
trigger: always_on
description: Handles:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PDFme is an open-source TypeScript-based PDF generation and manipulation library for web and Node.js applications. It provides a complete solution for creating, designing, and manipulating PDFs with a focus on performance, minimal dependencies, and ease of use.

## Claude Code Integration

### Triggering Claude Code
- Use `@claude` in GitHub issues, PR comments, or reviews to trigger Claude Code assistance
- Claude Code is configured via `.github/workflows/claude.yml` with appropriate permissions
- Best practices:
  - Be specific about the problem or task
  - Include relevant error messages or logs
  - Mention specific files or components when applicable

### GitHub Workflow Integration
- Claude Code automatically runs on issue comments, PR reviews, and new issues containing `@claude`
- Has read access to repository contents, pull requests, and issues
- Can assist with code analysis, debugging, and implementation suggestions

## Environment Requirements

### Node.js and Package Manager
- **Node.js**: Version 16 or higher (recommended: 18+ or 20+ for better performance)
- **npm**: Compatible with Node.js version (npm 8+ recommended)
- **Memory**: Minimum 4GB RAM, 8GB+ recommended for large PDF operations

### Required Development Tools
- **TypeScript**: For type checking and compilation
- **Vite+ (`vp`)**: Unified task runner used for install/run/lint/fmt
- **Oxlint/Oxfmt**: Native linting and formatting through `vp`
- **Vitest**: Testing framework with image snapshot support

### OS-Specific Considerations
- **Windows**: Use Git Bash or WSL for shell commands
- **macOS/Linux**: Standard terminal works fine
- **Memory limits**: Increase Node.js heap size for large PDFs: `node --max-old-space-size=8192`

## Common Development Commands

### Initial Setup and Build
```bash
npm install          # Install all dependencies
npm run build        # Build all packages in correct order
```

### Development Workflow
To work on packages with live reloading:
1. Run development mode in the packages you're working on:
   ```bash
   cd packages/[package-name] && npm run dev
   ```
2. Run the playground to test changes:
   ```bash
   cd playground && npm run dev  # Opens at localhost:5173
   ```

### Testing
```bash
npm run test                      # Run all tests
npm run test -w packages/ui -- -u # Update UI snapshot tests
# Run tests in specific package:
cd packages/[package-name] && npm run test
```

### Code Quality
```bash
npm run lint  # Run vp native lint
npm run fmt   # Format code with vp native fmt
```

### Building Individual Packages
```bash
npm run build -w packages/common    # Build @pdfme/common
npm run build -w packages/schemas   # Build @pdfme/schemas
npm run build -w packages/generator # Build @pdfme/generator
npm run build -w packages/ui        # Build @pdfme/ui
```

## Architecture and Code Structure

### Monorepo Structure
- **packages/common**: Core types, utilities, and shared logic
- **packages/pdf-lib**: Forked pdf-lib with custom modifications
- **packages/schemas**: Built-in field types (text, image, table, barcode, etc.)
- **packages/generator**: PDF generation from templates
- **packages/ui**: React components (Designer, Form, Viewer)
- **packages/manipulator**: PDF operations (merge, split, rotate)
- **packages/converter**: Format conversion utilities
- **playground**: Interactive development and testing environment
- **website**: Documentation site (Docusaurus)

### Key Architectural Patterns

#### 1. Plugin-Based Field System
Each field type (text, image, table, etc.) is a plugin with three components:
- `pdf`: Renders in the PDF using pdf-lib
- `ui`: Renders interactively in the browser
- `propPanel`: Configuration UI for the Designer

Location: `packages/schemas/src/[field-type]/index.ts`

#### 2. Template Structure
Templates consist of:
- `basePdf`: Either blank PDF with dimensions or custom PDF file
- `schemas`: 2D array where each sub-array represents a page
- `staticSchemas`: Optional fields that appear on every page

Type definitions: `packages/common/src/types.ts`

#### 3. Dynamic Layout Engine
Handles:
- Dynamic height calculation for expandable fields
- Automatic page breaking
- Layout tree management

Key function: `packages/generator/src/dynamicTemplate.ts:getDynamicTemplate`

#### 4. UI Component Hierarchy
All UI components extend `BaseUIClass` and support three modes:
- `viewer`: Read-only display
- `form`: Interactive input
- `designer`: Template creation

Base class: `packages/ui/src/class.ts`

#### 5. Expression System
Secure JavaScript expression evaluator for dynamic content:
- Uses Acorn for parsing
- AST validation for security
- Cached compilation

Implementation: `packages/common/src/expression.ts`

### Important Implementation Details

1. **Build Order**: Due to dependencies, packages must be built in order:
   pdf-lib → common → converter → schemas → parallel(generator, ui, manipulator)

2. **Font Management**: Custom fonts are loaded and cached in the UI components and embedded with subsetting in PDFs

3. **Validation**: Uses Zod schemas for runtime validation throughout the codebase


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pdfme/pdfme](https://github.com/pdfme/pdfme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
