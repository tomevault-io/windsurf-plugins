---
trigger: always_on
description: Handles $ref resolution and spec normalization. Uses different strategies based on OpenAPI version:
---

# CLAUDE.md - AI Assistant Guide for swagger-client

This document provides comprehensive guidance for AI assistants working on the swagger-client codebase.

## Project Overview

**swagger-client** is a JavaScript module that allows you to fetch, resolve, and interact with Swagger/OpenAPI documents. It supports:
- OpenAPI 3.2.0 (latest)
- OpenAPI 3.1.0
- OpenAPI 3.0.x (3.0.0 through 3.0.4)
- Swagger/OpenAPI 2.0
- Legacy Swagger 1.x (via version 2.x branch)

**Current Version**: 3.36.0

The library is used by Swagger-UI and other tools in the OpenAPI ecosystem to parse, resolve references, and execute operations defined in OpenAPI specifications.

## Repository Structure

```
swagger-client/
├── src/                    # Source code (ES6+ modules)
│   ├── execute/           # Request execution logic for OAS operations
│   │   ├── oas3/         # OpenAPI 3.x execution
│   │   └── swagger2/     # Swagger 2.0 execution
│   ├── helpers/          # Utility functions and helpers
│   ├── http/             # HTTP client implementation
│   │   └── serializers/  # Request/response serialization
│   ├── resolver/         # Spec resolution and $ref handling
│   │   ├── apidom/       # ApiDOM-based resolution (OAS 3.1)
│   │   ├── specmap/      # Legacy spec resolution engine
│   │   └── strategies/   # Version-specific resolution strategies
│   ├── subtree-resolver/ # Partial spec resolution
│   ├── index.js          # Main entry point
│   ├── interfaces.js     # Tags interface generation
│   ├── constants.js      # Global constants
│   └── commonjs.js       # CommonJS entry point
├── test/                  # Test suite
│   ├── bugs/             # Bug reproduction tests
│   ├── data/             # Test fixtures and sample specs
│   ├── execute/          # Execution tests
│   ├── helpers/          # Helper function tests
│   ├── http/             # HTTP client tests
│   ├── oas3/             # OpenAPI 3.x specific tests
│   ├── swagger2/         # Swagger 2.0 specific tests
│   ├── resolver/         # Resolution tests
│   └── build-artifacts/  # Build output verification tests
├── config/               # Build and test configuration
│   ├── jest/            # Jest test configurations
│   └── webpack/         # Webpack build configurations
├── docs/                 # Documentation
│   ├── usage/           # Usage documentation
│   ├── development/     # Development guides
│   └── migration/       # Migration guides
├── lib/                  # Built CommonJS output (gitignored)
├── es/                   # Built ES modules output (gitignored)
├── dist/                 # Built browser UMD bundle (gitignored)
└── .github/              # GitHub configuration
    └── workflows/       # CI/CD workflows
```

## Technology Stack

### Core Technologies
- **Language**: JavaScript (ES6+)
- **Build Tools**: Babel, Webpack
- **Test Framework**: Jest
- **Module Formats**: CommonJS, ES Modules, UMD (browser)

### Key Dependencies
- **@swagger-api/apidom-\***: ApiDOM suite for OpenAPI 3.1 parsing and resolution
- **ramda**: Functional programming utilities
- **js-yaml**: YAML parsing
- **deepmerge**: Deep object merging
- **fast-json-patch**: JSON Patch operations
- **node-abort-controller**: AbortController polyfill for Node.js
- **openapi-path-templating**: Path parameter templating
- **openapi-server-url-templating**: Server URL templating

### Development Dependencies
- **eslint**: Code linting (Airbnb base config)
- **prettier**: Code formatting
- **commitlint**: Commit message validation
- **husky**: Git hooks
- **lint-staged**: Pre-commit linting

## Build System

The project uses Babel for transpilation and Webpack for browser bundling.

### Build Targets
1. **UMD Browser Bundle** (`dist/swagger-client.browser.min.js`)
   - For `<script>` tag inclusion
   - Includes all polyfills
   - Minified and source-mapped

2. **CommonJS** (`lib/`)
   - ES5 code with CommonJS `require`/`module.exports`
   - For Node.js and older bundlers

3. **ES Modules** (`es/`)
   - ES5 code with ES6 `import`/`export`
   - For modern bundlers and tree-shaking

### Babel Configuration
The project uses environment-specific Babel configurations (see `babel.config.js`):
- `BABEL_ENV=commonjs`: CommonJS output
- `BABEL_ENV=es`: ES modules output
- `BABEL_ENV=browser`: Browser UMD bundle

### Browser/Node Polyfills
The project uses conditional imports for platform-specific code:
- `btoa.node.js` vs `btoa.browser.js`
- `fetch-polyfill.node.js` vs `fetch-polyfill.browser.js`
- `abortcontroller-polyfill.node.js` vs `abortcontroller-polyfill.browser.js`

These are configured in `package.json` under the `browser` field.

## Development Workflow

### Setup
```bash
# Use correct Node.js version (see .nvmrc)
nvm use

# Install dependencies
npm install

# Build all targets
npm run build

# Run tests
npm test
```

### Node.js Requirements
- **Development**: Node.js >= 22.11.0, npm >= 10.9.0
- **Runtime Support**: Node.js >= 22
- **Note**: EOL Node.js versions may be dropped without major version bump

### Available Scripts
- `npm run lint`: Check for linting errors
- `npm run lint:fix`: Auto-fix linting errors
- `npm run build`: Build all targets (UMD, CommonJS, ES)
- `npm run build:umd:browser`: Build browser bundle only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swagger-api/swagger-client](https://github.com/swagger-api/swagger-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
