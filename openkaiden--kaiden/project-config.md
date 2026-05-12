---
trigger: always_on
description: This file provides guidance to AI code assistants when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI code assistants when working with code in this repository.

## Project Overview

Kaiden is an Electron-based desktop application built with Svelte that provides AI-powered container and Kubernetes management capabilities. It integrates with multiple AI providers (Gemini, OpenAI-compatible services, OpenShift AI) and supports flow execution through providers like Goose. The application also implements the Model Context Protocol (MCP) for AI agent integration.

## Core Architecture

### Multi-Process Electron Application

Kaiden follows the standard Electron architecture with three main processes:

- **Main Process** (`packages/main`): Node.js backend handling system integration, extension loading, container/Kubernetes operations, and business logic
- **Renderer Process** (`packages/renderer`): Svelte-based UI running in the browser context
- **Preload Scripts** (`packages/preload` and `packages/preload-webview`): Bridge layer for secure IPC communication between main and renderer

### Plugin System and Dependency Injection

The core plugin system is implemented in `packages/main/src/plugin/index.ts` using Inversify for dependency injection. Key components include:

- **PluginSystem class**: Orchestrates extension loading, configuration registry, and all core services
- **Extension Loader** (`ExtensionLoader`): Manages extension lifecycle (loading, starting, stopping, uninstalling)
- **Container**: Inversify DI container binding all major registries and managers as singletons

All major services are registered as singletons in the DI container during initialization:

- `ProviderRegistry`: Manages inference, container, and Kubernetes providers
- `ContainerProviderRegistry`: Handles Docker/Podman container operations
- `KubernetesClient`: Kubernetes cluster management
- `MCPManager` and `MCPRegistry`: Model Context Protocol integration
- `FlowManager`: Manages flow execution with providers like Goose
- `ChatManager`: AI chat functionality
- `ConfigurationRegistry`: Settings and configuration management

### Extensions

Extensions are located in the `extensions/` directory and follow a standard structure:

- Each extension has a `package.json` with `main` pointing to `./dist/extension.js`
- Extensions must declare `engines.kaiden` version compatibility
- Extensions can contribute:
  - Inference providers (Gemini, OpenAI-compatible, OpenShift AI)
  - Flow providers (Goose)
  - MCP registries
  - Configuration properties
- Extensions are built using Vite and export a standard activation API

Available built-in extensions:

- `gemini`: Google Gemini AI provider integration
- `goose`: Goose flow execution provider
- `mcp-registries`: MCP server registries
- `openai-compatible`: OpenAI-compatible API support
- `openshift-ai`: OpenShift AI platform integration

### Extension API

Extensions interact with Kaiden through `@openkaiden/api` (`packages/extension-api`), which provides TypeScript definitions for:

- Provider registration (inference, container, Kubernetes, VM, flow)
- Configuration management
- Command and menu contributions
- UI components (webviews, views, status bar items)
- Authentication providers
- MCP server integration

## Development Commands

### Setup and Installation

```bash
# Install dependencies
pnpm install

# Start in watch/development mode
pnpm watch
```

### Building

```bash
# Build entire application
pnpm run build

# Build individual packages
pnpm run build:main        # Main process
pnpm run build:renderer    # UI/renderer
pnpm run build:preload     # Preload scripts
pnpm run build:extensions  # All extensions

# Build specific extension
cd extensions/gemini && pnpm run build
```

### Testing

```bash
# Run all tests (unit + e2e)
pnpm test

# Unit tests only
pnpm run test:unit

# Unit tests with coverage
pnpm run test:unit:coverage

# Run tests for specific packages
pnpm run test:main           # Main process tests
pnpm run test:renderer       # Renderer tests
pnpm run test:preload        # Preload tests
pnpm run test:extensions     # Extension tests

# E2E tests with Playwright
pnpm run test:e2e            # Build and run e2e tests
pnpm run test:e2e:run        # Run e2e tests only (must build first)
pnpm run test:e2e:report     # Show Playwright test report

# Watch mode for development
pnpm run test:watch
```

For detailed Playwright E2E testing guidance (Page Object Model, fixtures, locator conventions, examples), see `.agents/skills/playwright-testing/`.

For UI component development guidelines (color-registry usage, Icon component, reusable components), see `.agents/skills/ui-components/` and `CODE-GUIDELINES.md`.

### Code Quality

```bash
# Format code
pnpm run format:check        # Check formatting
pnpm run format:fix          # Fix formatting issues

# Linting
pnpm run lint:check          # Check for linting issues
pnpm run lint:fix            # Fix linting issues

# Type checking
pnpm run typecheck           # Check all packages
pnpm run typecheck:main      # Main process only
pnpm run typecheck:renderer  # Renderer only
pnpm run svelte:check        # Svelte component type checking
```

### Building Distributables

```bash
# Development build (directory output, no packaging)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openkaiden/kaiden](https://github.com/openkaiden/kaiden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
