---
trigger: always_on
description: This document provides context and instructions for AI agents (GitHub Copilot, Cursor, and other agentic platforms) working on the node-osc project.
---

# Agent Instructions for node-osc

This document provides context and instructions for AI agents (GitHub Copilot, Cursor, and other agentic platforms) working on the node-osc project.

## Project Overview

**node-osc** is a Node.js library for sending and receiving [Open Sound Control (OSC)](http://opensoundcontrol.org) messages over UDP. It provides a simple, no-frills API inspired by pyOSC.

### Key Features
- Send and receive OSC messages and bundles
- Dual module support (ESM and CommonJS)
- Both callback and async/await APIs
- TypeScript type definitions generated from JSDoc
- Well-tested with comprehensive test coverage
- Supports Node.js 20, 22, and 24+

## Architecture

### Core Components

1. **Server** (`lib/Server.mjs`) - EventEmitter-based OSC server for receiving messages
   - Listens on UDP socket
   - Emits events: `listening`, `message`, `bundle`, `error`, and address-specific events

2. **Client** (`lib/Client.mjs`) - OSC client for sending messages
   - Sends messages over UDP
   - Supports both callbacks and async/await

3. **Message** (`lib/Message.mjs`) - Represents a single OSC message
   - Contains address (string) and arguments (array)
   - Can append additional arguments

4. **Bundle** (`lib/Bundle.mjs`) - Represents a collection of OSC messages
   - Contains timetag and array of elements (messages or nested bundles)
   - Used for sending multiple messages together

5. **Low-level encoding/decoding** (`lib/osc.mjs`, `lib/internal/`) - Binary OSC protocol implementation
   - `encode()` - Converts Message/Bundle objects to binary Buffer
   - `decode()` - Parses binary Buffer into Message/Bundle objects

### Module System

The project uses **ESM as the source format** but provides **dual ESM/CommonJS support**:
- Source files: `lib/**/*.mjs` (ESM)
- Built CommonJS files: `dist/lib/**/*.js` (transpiled via Rollup)
- Generated ESM TypeScript definitions: `types/*.d.mts` (with `types/index.d.mts` as the ESM type entry point)
- Generated CommonJS TypeScript definitions: `dist/types/*.d.ts` (generated from the Rollup CommonJS output, with `dist/types/index.d.ts` as the CJS type entry point)

**Important:**
- `types/` contains generated declarations for the ESM source tree in `lib/`
- `dist/types/` contains generated declarations for the CommonJS build in `dist/lib/`
- Do not manually edit generated declaration files in either location

### Package Exports

```json
{
  "types": "./types/index.d.mts",
  "exports": {
    ".": {
      "import": {
        "types": "./types/index.d.mts",
        "default": "./lib/index.mjs"
      },
      "require": {
        "types": "./dist/types/index.d.ts",
        "default": "./dist/lib/index.js"
      },
      "default": "./lib/index.mjs"
    }
  }
}
```

## Development Workflow

### Essential Commands

```bash
# Install dependencies
npm install

# Run linter (ESLint)
npm run lint

# Build the project (clean, transpile to CJS, generate types)
npm run build

# Run all tests (lint + build + ESM tests + CJS tests)
npm test

# Run only ESM tests
npm run test:esm

# Run only CJS tests
npm run test:cjs

# Generate API documentation from JSDoc
npm run docs

# Clean build artifacts
npm run clean
```

### Testing Strategy

- Tests are written in ESM format in `test/test-*.mjs`
- Tests are run against both ESM source (`lib/`) and transpiled CJS (`dist/`)
- Uses `tap` test framework
- Tests typically use ephemeral ports via `new Server(0, ...)` and wait for readiness with `once(server, 'listening')`
- Always run `npm run build` before running CJS tests
- **100% test coverage is required** - All lines, branches, functions, and statements must be covered

### Build Process

1. **Clean**: Removes `dist/` and `types/` directories
2. **Rollup**: Transpiles ESM to CommonJS in `dist/` directory
3. **TypeScript (ESM declarations)**: Generates `.d.mts` type definitions from JSDoc in `types/` using the ESM source tree in `lib/`
4. **TypeScript (CJS declarations)**: Generates `.d.ts` type definitions in `dist/types/` from the Rollup-generated CommonJS tree in `dist/lib/`

The build is automatically run before publishing (`prepublishOnly` script).

## Coding Standards

### JavaScript Style

- **ES Modules**: Use ESM syntax (`import`/`export`)
- **File extension**: Use `.mjs` for ESM files
- **Linting**: Follow ESLint rules in `eslint.config.mjs`
- **Modern JavaScript**: Use async/await, arrow functions, destructuring
- **Error handling**: Always handle errors in async operations

### Documentation

- **JSDoc comments**: All public APIs must have JSDoc comments
- **Type annotations**: Use JSDoc types for TypeScript generation
- **Examples**: Include code examples in JSDoc comments
- **Auto-generated docs**: Run `npm run docs` after changing JSDoc comments; update `scripts/generate-docs.mjs` if the API doc layout or anchor generation needs to change

Example JSDoc pattern:
```javascript
/**
 * Sends an OSC message or bundle.
 * 
 * @param {Message|Bundle|string} msg - The message, bundle, or address to send.
 * @param {...*} args - Additional arguments (used when first param is a string address).
 * @returns {Promise<void>}
 * 
 * @example
 * await client.send('/test', 123);
 * 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MylesBorins/node-osc](https://github.com/MylesBorins/node-osc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
