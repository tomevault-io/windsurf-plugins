---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is TDI2 (TypeScript Dependency Injection Attempt #2) - a React Service Injection (RSI) framework that enables enterprise-grade architecture for React applications. The project moves state and logic out of components into reactive services, eliminating prop drilling and providing automatic state synchronization.

## Repository Structure

This is a monorepo with the following structure:

- **`/monorepo/`** - Main monorepo containing packages and apps
- **`/examples/`** - Example applications demonstrating TDI2 usage
- **`/monorepo/apps/docs-starlight/`** - Comprehensive Starlight-based documentation site
- **`/monorepo/apps/di-debug/`** - React-based debug tools with DI analytics CLI and web dashboard
- **`/github-issue-sync/`** - Issue synchronization tooling

## Common Commands

### Development Commands (from monorepo root)
```bash
cd monorepo
bun install           # Install dependencies
bun run dev           # Start development mode for all apps
bun run build         # Build all packages and apps
bun run test          # Run all tests
bun run lint          # Run linting
bun run check-types   # Run TypeScript type checking
```

### Package-specific Commands
```bash
# Core DI package
cd monorepo/packages/di-core
bun test              # Run core DI tests
bun run build         # Build the core package

# Vite plugin
cd monorepo/packages/vite-plugin-di
bun test              # Run plugin tests
bun run build         # Build the plugin

# Examples
cd examples/tdi2-basic-example
npm run dev           # Start example app
npm run build         # Build example

# DI Debug Tools
cd monorepo/apps/di-debug
bun run dev           # Start React dashboard development
bun run build         # Build React dashboard
bun run tdi2          # Run CLI commands (analyze, serve, etc.)
```

### Testing Commands
```bash
# Run specific tests
bun test functional-di-enhanced-transformer.test.ts
bun test integrated-interface-resolver.test.ts

# Interactive test selection (from di-core)
bun run test:io
```

## Debugging & Logging

TDI2 uses the `debug` package for structured, namespace-based logging controlled via the `DEBUG` environment variable.

### Enable Logs

Set the DEBUG environment variable to control which logs appear:

```bash
# All TDI2 logs (most verbose)
DEBUG=* bun run dev

# All di-core transformation logs
DEBUG=di-core:* bun run dev

# All vite-plugin logs
DEBUG=vite-plugin-di:* bun run dev

# Specific module only
DEBUG=di-core:config-manager bun run dev

# Multiple specific modules
DEBUG=di-core:config-manager,vite-plugin-di:plugin bun run dev
```

### Available Namespaces

**Vite Plugin:**
- `vite-plugin-di:plugin` - Vite plugin operations

**Plugin Core:**
- `plugin-core:transform-orchestrator` - Transformation orchestration

**DI Core Tools:**
- `di-core:config-manager` - Configuration generation
- `di-core:functional-transformer` - Functional component transformation
- `di-core:transformation-pipeline` - Transformation pipeline
- `di-core:enhanced-transformer` - Enhanced DI transformation
- `di-core:interface-resolver` - Interface resolution
- `di-core:enhanced-service-validator` - Service validation
- `di-core:enhanced-interface-extractor` - Interface extraction
- `di-core:enhanced-dependency-extractor` - Dependency extraction
- `di-core:recursive-inject-extractor` - Recursive inject extraction
- `di-core:shared-service-registry` - Service registry
- `di-core:shared-dependency-extractor` - Dependency extraction
- `di-core:property-access-updater` - Property access updates
- `di-core:import-manager` - Import management
- `di-core:config-processor` - Configuration processing
- `di-core:debug-file-generator` - Debug file generation

### For Contributors

When adding logging to new modules:

```typescript
import { consoleFor } from '../logger';  // or '@tdi2/di-core/tools'
const console = consoleFor('di-core:your-module-name');

// Always shown with namespace prefix (errors/warnings)
console.error('❌ Error message');      // Outputs: [di-core:your-module-name] ❌ Error message
console.warn('⚠️  Warning message');    // Outputs: [di-core:your-module-name] ⚠️  Warning message

// Only shown with DEBUG=di-core:your-module-name (or DEBUG=di-core:*)
console.log('Info message');
console.debug('Detailed debug information');
```

**Benefits:**
- All log output includes namespace prefix for transparency
- Errors and warnings are always shown (even without DEBUG)
- Debug logs controlled via DEBUG environment variable
- Easy to trace which module produced each message

**DO NOT** use `if (verbose)` checks - the DEBUG environment variable provides granular control.

## Core Architecture

### TDI2 System Components

1. **DI Core (`@tdi2/di-core`)** - The dependency injection container and decorators
2. **Vite Plugin (`@tdi2/vite-plugin-di`)** - Compile-time code transformation
3. **Functional DI Transformer** - Converts components to use service injection
4. **Interface Resolution System** - Automatically resolves TypeScript interfaces to implementations

### Key Technologies

- **Valtio** - Reactive state management via proxies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [7frank/tdi2](https://github.com/7frank/tdi2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
