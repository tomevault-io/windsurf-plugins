---
trigger: always_on
description: **BEFORE making ANY changes to configuration options or API methods, read this:**
---

# Copilot Instructions for JustGage

## 🚨 CRITICAL: Documentation Synchronization Requirement

**BEFORE making ANY changes to configuration options or API methods, read this:**

JustGage uses a **triple-synchronized documentation system**. When you change ANYTHING related to configuration options or API methods, you MUST update these THREE files simultaneously:

1. **`src/types/index.d.ts`** - TypeScript interface definitions
2. **`README.md`** - Configuration options table
3. **`docs/src/views/DocsView.vue`** - `configOptions` array

**Failure to keep these synchronized will break:**

- TypeScript compilation
- User documentation
- Interactive documentation site
- Developer experience

**See the "Configuration Management & Documentation Maintenance" section below for detailed protocols.**

## Project Overview

JustGage is a handy JavaScript plugin for generating and animating nice & clean dashboard gauges. This repository contains:

- **v2.0+ (Modern)**: ES6+ modules with native SVG APIs (located in `/src/`)
- **v1.x (Legacy)**: RaphaelJS-based implementation (reference at `/docs/public/justgage.js`)
- **Documentation**: Vue 3 + Vuetify 3 website (located in `/docs/`)
- **Migration Tools**: Utilities to help users migrate from v1.x to v2.0+ (located in `/migration-utils/`)

## Architecture & Code Standards

### Modern Implementation (v2.0+)

- **Language**: TypeScript/JavaScript ES6+
- **Module System**: ES6 modules with UMD build
- **SVG**: Native SVG APIs (no external dependencies)
- **Build Tool**: ESBuild for fast compilation
- **Testing**: Node.js native test runner with jsdom for DOM testing

### Legacy Reference (v1.x)

- **Location**: `/docs/public/justgage.js` (1728 lines)
- **Dependencies**: RaphaelJS for SVG manipulation
- **Architecture**: UMD module pattern
- **Use Case**: Reference for feature parity and migration compatibility

### Documentation Site

- **Framework**: Vue 3 + Composition API
- **UI Library**: Vuetify 3 (Material Design)
- **Features**:
  - Interactive playground with real-time gauge rendering
  - Comprehensive API documentation
  - Migration guides from v1.x to v2.0+
  - Mobile-responsive design with navigation drawers

## Development Guidelines

### Code Style

- Follow **ESLint** configuration (see `eslint.config.js`)
- Use **Prettier** for formatting (see `.prettierrc`)
- Write **TypeScript** with strict type checking
- Use **conventional commit messages** (see Commit Standards below)

### Testing

- Write unit tests for all new features
- Maintain backward compatibility with v1.x API where possible
- Test cross-browser compatibility
- Include performance tests for animation-heavy features

### File Structure

```
/src/                 # Modern v2.0+ implementation
/docs/public/         # Contains legacy v1.x reference (justgage.js)
/docs/               # Vue 3 + Vuetify 3 documentation site
/migration-utils/     # Tools for v1.x to v2.0+ migration
/tests/               # Test suites
/dist/                # Built distribution files
```

## Key Features to Maintain

### Core Gauge Functionality

- **Value Display**: Numeric value with customizable formatting
- **Range Configuration**: Min/max values with validation
- **Animation**: Smooth transitions and initial animations
- **Responsive Design**: Relative gauge sizing for different containers
- **Color Schemes**: Level-based colors and custom gradients
- **Labels & Titles**: Customizable text elements

### Advanced Features

- **Custom Sectors**: Color-coded value ranges
- **Pointer Customization**: Different pointer styles and sizes
- **Event Hooks**: Animation start/end callbacks
- **Value Formatting**: Custom number formatting and symbols
- **Accessibility**: ARIA labels and keyboard navigation
- **Performance**: Optimized for multiple gauges on single page

## API Compatibility

### Constructor Pattern

```javascript
// v2.0+ (Modern)
import JustGage from 'justgage';
const gauge = new JustGage({
  id: 'gauge-container',
  value: 67,
  min: 0,
  max: 100,
});

// v1.x (Legacy Reference)
var gauge = new JustGage({
  id: 'gauge-container',
  value: 67,
  min: 0,
  max: 100,
});
```

### Core Methods

- `refresh(value, max, min, label)` - Update gauge values
- `update(options)` - Update configuration options
- `destroy()` - Clean up and remove gauge

## Migration Considerations

### Breaking Changes in v2.0+

- **No RaphaelJS dependency** - Uses native SVG APIs
- **ES6 modules** - Requires build step or modern browser
- **TypeScript support** - Full type definitions included
- **Performance improvements** - Optimized rendering pipeline

### Compatibility Layer

- Maintain same API surface for easy migration
- Provide migration utilities in `/migration-utils/`
- Document breaking changes clearly
- Offer codemods where possible

## Documentation Standards

### Code Examples

- Always provide both v1.x and v2.0+ examples
- Include TypeScript examples where relevant
- Show responsive and accessibility best practices
- Demonstrate error handling

### API Documentation

- Document all configuration options with types
- Include default values and valid ranges
- Provide visual examples in playground
- Show migration paths from v1.x

## Commit Standards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [toorshia/justgage](https://github.com/toorshia/justgage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
