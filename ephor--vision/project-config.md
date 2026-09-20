---
trigger: always_on
description: > **Last Updated:** 2026-01-24
---

# CLAUDE.md - AI Assistant Guide for Vision

> **Last Updated:** 2026-01-24
> **Repository:** Vision - Universal Observability Dashboard for API Development

This document provides comprehensive guidance for AI assistants working with the Vision codebase. It covers architecture, conventions, workflows, and best practices.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Repository Structure](#repository-structure)
3. [Development Environment](#development-environment)
4. [Key Architectural Concepts](#key-architectural-concepts)
5. [Code Conventions](#code-conventions)
6. [Development Workflows](#development-workflows)
7. [Testing Guidelines](#testing-guidelines)
8. [Common Tasks](#common-tasks)
9. [Package-Specific Notes](#package-specific-notes)
10. [Troubleshooting](#troubleshooting)

---

## Project Overview

**Vision** is a development dashboard that provides unified observability across protocols and validation libraries. It's designed to add monitoring, tracing, and debugging capabilities to existing Express, Fastify, or Hono applications, or be used as a standalone meta-framework.

### Core Features
- Multi-protocol support (REST, with GraphQL, tRPC, MCP in development)
- Universal validation integration (Zod, Valibot, Standard Schema v1)
- Real-time tracing and logging via WebSocket
- API playground with multi-tab testing
- Automatic request template generation
- "Wide Events" logging philosophy (add context once, see it everywhere)

### Technology Stack
- **Language:** TypeScript (strict mode)
- **Package Manager:** Bun 1.3.0
- **Monorepo Tool:** Turborepo 2.5.9+
- **Test Framework:** Bun Test
- **Build Tool:** TypeScript Compiler (tsc)
- **Frameworks:** Hono, Express, Fastify
- **Validation:** Zod, Valibot (peer dependencies)
- **WebSocket:** ws library with JSON-RPC 2.0

---

## Repository Structure

```
vision/
├── packages/               # Core packages
│   ├── core/              # Core tracing, validation, WebSocket server
│   ├── server/            # Meta-framework built on Hono
│   ├── adapter-express/   # Express.js adapter
│   ├── adapter-fastify/   # Fastify adapter
│   ├── adapter-hono/      # Hono adapter
│   ├── ui/                # Shared React components
│   ├── eslint-config/     # Shared ESLint configuration
│   └── typescript-config/ # Shared TypeScript configurations
├── apps/                  # Applications
│   ├── web/               # Dashboard web app (Vite + React)
│   └── docs/              # Documentation site
├── examples/              # Example applications
│   ├── express/           # Express example
│   ├── fastify/           # Fastify example
│   ├── hono/              # Hono example
│   └── vision/            # Vision Server example
├── .changeset/            # Changesets for versioning
├── .github/workflows/     # CI/CD workflows
└── [config files]         # Root configuration files
```

### Key Configuration Files
- `package.json` - Root workspace configuration
- `turbo.json` - Turborepo build configuration
- `bun.lock` - Dependency lock file
- `.eslintrc.js` - ESLint configuration
- `.npmrc` - NPM configuration (auto-install-peers = true)
- `.gitignore` - Git ignore patterns

---

## Development Environment

### Prerequisites
- **Bun:** 1.3.0 (required, specified in `packageManager` field)
- **Node.js:** 20+ (for compatibility)
- **Git:** For version control

### Initial Setup

```bash
# Clone repository
git clone https://github.com/ephor/vision.git
cd vision

# Install dependencies
bun install --frozen-lockfile

# Build all packages
bun run build

# Run tests
bun run test

# Start example application
bun example:hono
```

### Available Scripts

```bash
# Build & Development
bun run build          # Build all packages (Turbo)
bun run dev            # Run all packages in dev mode
bun run test           # Run all tests
bun run lint           # Lint all packages
bun run format         # Format code with Prettier

# Examples
bun example:hono       # Run Hono example
bun example:express    # Run Express example
bun example:fastify    # Run Fastify example
bun example:server     # Run Vision Server example

# Documentation
bun docs:dev           # Run docs in dev mode
bun docs:build         # Build docs

# Versioning & Release
bun run changeset      # Create a changeset
bun run ci:version     # Version packages
bun run release        # Build and publish packages
```

---

## Key Architectural Concepts

### 1. Monorepo with Turborepo

Vision uses **Turborepo** for efficient builds with dependency graph awareness:

- **Build caching:** Builds are cached based on inputs
- **Parallel execution:** Independent tasks run concurrently
- **Dependency tracking:** Packages build in topological order
- **Workspace protocol:** Internal dependencies use `workspace:*`

**Turborepo Configuration (`turbo.json`):**
```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", "out/**", ".next/**"]
    },
    "dev": { "cache": false, "persistent": true },
    "test": { "cache": false, "persistent": true }
  }
}
```

### 2. Core Architecture (@getvision/core)

The `@getvision/core` package is the foundation, providing:

#### **Tracing System**
- **TraceStore:** In-memory circular buffer for traces (configurable max size)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ephor/vision](https://github.com/ephor/vision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
