---
trigger: always_on
description: > **Last Updated:** 2026-02-24
---

# CLAUDE.md - Swagger UI Codebase Guide

> **Last Updated:** 2026-02-24
> **Version:** 5.32.0 (in development)
> **Purpose:** Comprehensive guide for AI assistants working with the Swagger UI codebase

---

## Table of Contents

1. [Repository Overview](#repository-overview)
2. [Project Architecture](#project-architecture)
3. [Development Setup](#development-setup)
4. [Build System](#build-system)
5. [Testing Infrastructure](#testing-infrastructure)
6. [Code Style & Conventions](#code-style--conventions)
7. [Git Workflow](#git-workflow)
8. [Plugin Architecture](#plugin-architecture)
9. [Key Files & Directories](#key-files--directories)
10. [Common Workflows](#common-workflows)
11. [Important Guidelines](#important-guidelines)

---

## Repository Overview

### What is Swagger UI?

Swagger UI is a tool that allows developers to visualize and interact with API resources without having implementation logic in place. It's automatically generated from OpenAPI (formerly Swagger) Specification documents.

### Multi-Package Monorepo Structure

This repository publishes **three different npm packages**:

1. **swagger-ui** (main package)
   - Traditional npm module for single-page applications
   - Entry: `dist/swagger-ui.js`
   - ES Module: `dist/swagger-ui-es-bundle-core.js`
   - Includes dependency resolution via Webpack/Browserify

2. **swagger-ui-dist** (distribution package)
   - Dependency-free module for server-side projects
   - Published separately via GitHub workflow
   - Template location: `swagger-ui-dist-package/`

3. **swagger-ui-react** (React component)
   - React wrapper component
   - Location: `flavors/swagger-ui-react/`
   - Uses React hooks
   - Released separately via GitHub workflow

### OpenAPI Specification Compatibility

- **Current Support:** OpenAPI 2.0, 3.0.x, 3.1.x
- **Latest Version:** v5.31.0 (supports up to OpenAPI 3.1.2)

### License

Apache 2.0 - See LICENSE and NOTICE files for details.

---

## Project Architecture

### Technology Stack

**Core Framework:**
- React 18 (>=16.8.0 <20) - UI components
- Redux 5.0.1 - State management
- Redux Immutable 4.0.0 - Immutable state
- Immutable.js 3.x - Immutable data structures
- React Redux 9.2.0 - React-Redux bindings

**API & Schema Processing:**
- swagger-client 3.36.0 - OpenAPI client
- js-yaml 4.1.1 - YAML parsing
- remarkable 2.0.1 - Markdown rendering

**Security:**
- DOMPurify 3.2.6 - HTML sanitization (CRITICAL for XSS prevention)
- serialize-error 8.1.0 - Error serialization

**Build Tools:**
- Webpack 5.97.1 - Module bundling
- Babel 7.26.x - JavaScript transpilation
- sass-embedded 1.86.0 - SCSS compilation
- PostCSS - CSS processing

**Testing:**
- Jest 29.7.0 - Unit testing
- Cypress 14.2.0 - E2E testing
- Enzyme 3.11.0 - React component testing

**Development:**
- ESLint 8.57.0 - JavaScript linting
- Prettier 3.5.3 - Code formatting
- Stylelint 16.19.1 - CSS linting
- Husky 9.1.7 - Git hooks
- lint-staged 15.5.0 - Pre-commit linting

### Plugin-Based Architecture

Swagger UI uses a **sophisticated plugin system** powered by Redux. The core system (`src/core/system.js`) manages:

- Plugin registration and lifecycle
- Redux store creation and middleware
- State plugin combination
- Action/selector binding
- Configuration management

**26 Core Plugins** (in `src/core/plugins/`):
- `auth` - Authentication handling
- `configs` - Configuration management
- `deep-linking` - URL-based navigation
- `download-url` - Spec downloading
- `err` - Error handling and transformation
- `filter` - API filtering
- `icons` - Icon components
- `json-schema-2020-12` - JSON Schema 2020-12 support
- `json-schema-2020-12-samples` - Sample generation
- `json-schema-5` - JSON Schema Draft 5 support
- `json-schema-5-samples` - Sample generation for Draft 5
- `layout` - Layout system
- `logs` - Logging
- `oas3` - OpenAPI 3.0.x support
- `oas31` - OpenAPI 3.1.x support
- `oas32` - OpenAPI 3.2.x support
- `on-complete` - Completion callbacks
- `request-snippets` - Code snippet generation
- `safe-render` - Safe component rendering
- `spec` - Specification handling
- `swagger-client` - API client integration
- `syntax-highlighting` - Code highlighting
- `util` - Utilities
- `versions` - Version detection
- `view` - View rendering
- `view-legacy` - Legacy view support

---

## Development Setup

### Prerequisites

- **Node.js:** >=24.18.0 (Node 24.x recommended, as defined in `.nvmrc`)
- **npm:** >=11.16.0
- **Git:** Any version
- **JDK 7+:** Required for Nightwatch.js integration tests

### Installation Steps

```bash
# Clone the repository
git clone https://github.com/swagger-api/swagger-ui.git
cd swagger-ui

# Install dependencies
npm install

# Initialize Husky (optional, for git hooks)
npx husky init

# Start development server
npm run dev

# Open http://localhost:3200/
```

### Development Server

The `npm run dev` command starts a hot-reloading Webpack dev server on **port 3200**.

### Using Local API Definitions

Edit `dev-helpers/dev-helper-initializer.js` to change the spec URL:

```javascript
// Replace
url: "https://petstore.swagger.io/v2/swagger.json",

// With
url: "./examples/your-local-api-definition.yaml",
```

**Important:** Local files must be in the `dev-helpers/` directory or subdirectory. Use `dev-helpers/examples/` (already in `.gitignore`).

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swagger-api/swagger-ui](https://github.com/swagger-api/swagger-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
