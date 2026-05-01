---
trigger: always_on
description: <!-- nx configuration start-->
---

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

# General Guidelines for working with Nx

- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- You have access to the Nx MCP server and its tools, use them to help the user
- When answering questions about the repository, use the `nx_workspace` tool first to gain an understanding of the workspace architecture where applicable.
- When working in individual projects, use the `nx_project_details` mcp tool to analyze and understand the specific project structure and dependencies
- For questions around nx configuration, best practices or if you're unsure, use the `nx_docs` tool to get relevant, up-to-date docs. Always use this instead of assuming things about nx configuration
- If the user needs help with an Nx configuration or project graph error, use the `nx_workspace` tool to get any errors

<!-- nx configuration end-->

# Native Federation Core - AI Assistant Guide

## Project Overview

**Native Federation** is a browser-native implementation of the Module Federation mental model for building Micro Frontends. Unlike webpack Module Federation, this library is **framework-agnostic** and **build-tool-agnostic**, leveraging browser-native technologies like **ES Modules** and **Import Maps**.

### Key Concepts

- **Remote**: A separately built and deployed application that exposes ES modules for consumption by other apps
- **Host**: An application (shell) that loads remotes on demand at runtime
- **Shared Dependencies**: Libraries shared between host and remotes to avoid duplicate downloads
- **Import Maps**: Browser-native technology used to map module specifiers to URLs
- **remoteEntry.json**: Metadata file generated during build containing information about exposed modules and shared dependencies

### Technology Stack

- Written in TypeScript with ES Modules (`.js` extensions in imports)
- Uses esbuild for bundling (via adapters)
- Built with Nx monorepo tooling
- Uses pnpm workspaces
- Testing with Vitest (including browser tests with Playwright)

## Repository Structure

This is an **Nx monorepo** with three main packages:

```
packages/
├── core/              → @softarc/native-federation
├── runtime/           → @softarc/native-federation-runtime
└── node/              → @softarc/native-federation-node
```

### Package Breakdown

#### 1. `@softarc/native-federation` (core)

**Purpose**: Build-time functionality for configuring and bundling federated applications

**Key exports** (via `package.json` exports):

- `.` - Main build API: `federationBuilder`, `buildForFederation`, etc.
- `./config` - Configuration utilities: `withNativeFederation`, `shareAll`, `share`
- `./domain` - Type definitions and contracts
- `./internal` - Internal utilities (generally not for public use)

**Core responsibilities**:

- Parse and normalize federation configuration from `federation.config.js`
- Determine externals (dependencies that should not be bundled with main app)
- Bundle shared dependencies separately for caching
- Bundle exposed modules (remote entry points)
- Handle shared mappings (monorepo-internal libraries)
- Generate `remoteEntry.json` metadata files
- Manage federation cache for performance
- Support both browser and Node.js platforms

**Important files**:

- `src/lib/core/federation-builder.ts` - Main API: `init()`, `build()`, `close()`
- `src/lib/core/build-for-federation.ts` - Initial build orchestration
- `src/lib/core/rebuild-for-federation.ts` - Incremental rebuild logic (watch mode)
- `src/lib/core/bundle-shared.ts` - Bundles shared dependencies
- `src/lib/core/bundle-exposed-and-mappings.ts` - Bundles exposed modules
- `src/lib/core/build-adapter.ts` - Adapter pattern for build tools (esbuild, rollup, etc.)
- `src/lib/config/with-native-federation.ts` - Configuration normalization
- `src/lib/config/share-utils.ts` - `shareAll()`, `share()` helpers
- `src/lib/config/remove-unused-deps.ts` - Tree-shakes unused shared dependencies

**Build Adapter Pattern**:
The core library is build-tool agnostic. It expects a `NFBuildAdapter` that implements bundling logic. Reference implementations use esbuild (see `@softarc/native-federation-esbuild` package, separate repo).

#### 2. `@softarc/native-federation-runtime` (runtime)

**Purpose**: Runtime functionality for loading federated modules in the browser

**Key exports**:

- `initFederation()` - Initialize federation runtime, load remoteEntry.json files
- `loadRemoteModule()` - Dynamically load a module from a remote
- `watchFederationBuildCompletion()` - Watch for remote build notifications during development

**Core responsibilities**:

- Fetch and parse `remoteEntry.json` metadata from host and remotes
- Construct ES Module import maps with proper scoping
- Inject import maps into the DOM (uses `es-module-shims` polyfill)
- Manage version matching for shared dependencies
- Handle remote module loading with proper error handling
- Support hot-reload / live-reload during development

**Important files**:

- `src/lib/init-federation.ts` - Main entry point for federation initialization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [native-federation/native-federation-core](https://github.com/native-federation/native-federation-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
