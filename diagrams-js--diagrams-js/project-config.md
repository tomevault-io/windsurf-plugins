---
trigger: always_on
description: **diagrams-js** is a TypeScript port of the Python [diagrams](https://github.com/mingrammer/diagrams) library. It allows developers to draw cloud system architecture diagrams as code using TypeScript/JavaScript.
---

# diagrams-js - AI Agent Development Guide

## Project Overview

**diagrams-js** is a TypeScript port of the Python [diagrams](https://github.com/mingrammer/diagrams) library. It allows developers to draw cloud system architecture diagrams as code using TypeScript/JavaScript.

### Key Features

- **Cross-platform**: Works in browsers, Node.js, Deno and Bun
- **Graphviz-powered**: Uses WebAssembly-based Graphviz for rendering
- **17 Providers**: 2000+ node classes for AWS, Azure, GCP, Kubernetes, and more
- **Custom Nodes**: Support for external icons and images
- **Type-safe**: Full TypeScript support
- **Tree-shakable**: Import only what you need

### Architecture

```
src/
├── index.ts           # Main exports
├── Diagram.ts         # Core diagram class
├── Node.ts           # Base node class
├── Edge.ts           # Edge/connection class
├── Cluster.ts        # Cluster/grouping class
├── Custom.ts         # Custom nodes with external icons
├── icons.ts          # Icon injection utilities
├── types.ts          # TypeScript types
├── context.ts        # Context management
├── json.ts           # JSON serialization
├── plugins/          # Plugin system
│   ├── built-in/     # Built-in plugins (json, svg)
│   ├── registry.ts   # Plugin registry
│   ├── types.ts      # Plugin types
│   └── index.ts      # Plugin exports
└── providers/        # Auto-generated provider classes
    ├── aws/
    ├── azure/
    ├── gcp/
    └── ... (17 providers)

docs/                  # Docusaurus documentation
skills/                # AI agent skill definitions
tests/                 # Test suite
scripts/               # Build and generation scripts
```

## Technology Stack

- **Runtime**: TypeScript 5.0+
- **Build Tool**: Vite+ (unified toolchain)
- **Testing**: Vitest (via Vite+)
- **Linting**: Oxlint (via Vite+)
- **Formatting**: Oxfmt (via Vite+)
- **Rendering**: @viz-js/viz (Graphviz WASM)
- **Documentation**: Docusaurus

## AI Agent Skills

The `skills/` directory contains structured skill definitions for AI agents. Load relevant skills before working on related tasks:

- **getting-started** — Installation, first diagram, basic concepts
- **diagram-configuration** — Direction, theme, curve style, custom attributes
- **rendering-export** — SVG/PNG/JPG/DOT output, data URLs, file saving
- **json-serialization** — Export/import diagrams as JSON
- **svg-serialization** — Export/import diagrams as SVG with embedded metadata
- **provider-nodes** — Using 2000+ pre-built cloud provider nodes
- **custom-nodes** — External icons and images
- **node-connections** — `.to()`, `.from()`, `.with()`, `Edge()` styling
- **clusters-grouping** — Nested clusters and visual organization
- **diagrams-js-plugin-system** — Creating custom import/export plugins
- **creating-plugins** — Package structure, API, best practices
- **diagram-diff** — Comparing diagram versions
- **browser-integration** — CDN, bundlers, DOM insertion
- **nodejs-integration** — File system, sharp, local icons
- **python-migration** — Converting from Python diagrams library

## Development Workflow

> **Platform Note**: This is a Windows machine using PowerShell. Use PowerShell-compatible commands.

### Prerequisites

- Node.js 18+
- pnpm (preferred package manager)

### Essential Commands

```powershell
# Install dependencies
vp install

# Run tests
vp test

# Check code (format, lint, types)
vp check

# Fix auto-fixable issues
vp check --fix

# Build the library
vp run build

# Build providers (generates JS from TS)
vp run build:providers

# Full build (core + providers + fix)
vp run build

# Development mode
vp dev
```

### Testing

All tests must pass before committing:

```bash
vp test
```

Test files are in `tests/` directory using Vitest.

## Code Guidelines

### Style

- Use TypeScript strict mode
- Prefer explicit types over implicit
- Use `const` and `let`, never `var`
- Use arrow functions for callbacks
- Use async/await for async operations

### Naming Conventions

- Classes: PascalCase (e.g., `Diagram`, `CustomNode`)
- Functions: camelCase (e.g., `renderDiagram`)
- Constants: UPPER_SNAKE_CASE for true constants
- Files: kebab-case (e.g., `custom-node.ts`)
- Private methods: prefix with underscore (e.g., `_privateMethod`)

### Import/Export Style

- Use `.js` extension in imports (e.g., `import { X } from "./file.js"`)
- Use named exports preferred over default exports
- Group imports: external libs first, then internal modules

### Error Handling

- Always handle Promise rejections
- Use specific error types when possible
- Provide helpful error messages
- Use `void` operator for floating promises: `void promise`

### Comments

- Use JSDoc for public APIs
- Keep comments concise and relevant
- Update comments when code changes

## Project Structure Guidelines

### Adding New Features

1. **Core functionality**: Add to appropriate file in `src/`
2. **New providers**: Modify `scripts/generate.ts` and regenerate
3. **Tests**: Add to `tests/index.test.ts` or create new test files
4. **Documentation**: Update files in `docs/docs/`
5. **Skills**: Update or add skill files in `skills/` when behavior changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diagrams-js/diagrams-js](https://github.com/diagrams-js/diagrams-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
