---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

laf is an open-source serverless cloud development platform that provides cloud functions, cloud databases, and cloud storage out of the box. It enables developers to focus on business development without worrying about server management.

The platform consists of:

- **Web IDE**: Browser-based development environment for writing cloud functions
- **Cloud Functions**: Serverless function execution (Node.js/Python)
- **Cloud Database**: MongoDB-based database with access control
- **Cloud Storage**: S3-compatible object storage (MinIO)
- **Website Hosting**: Static site deployment

## Repository Structure

This is a monorepo managed by Lerna. Key workspaces:

### Core Services

- **`server/`** - NestJS-based API server that manages the entire laf platform

  - Handles auth, applications, functions, database, storage, logs, billing, domains, certificates, and metrics
  - Uses MongoDB for system database
  - Runs on port 3000

- **`runtimes/nodejs/`** - Node.js runtime engine that executes user cloud functions

  - Express-based runtime that executes cloud functions
  - Provides database access proxy
  - Includes TypeScript language server for IDE support
  - Each application instance runs its own runtime pod in Kubernetes

- **`web/`** - React-based web console UI
  - Built with Vite, React 18, Chakra UI, TailwindCSS
  - Monaco editor integration for code editing
  - Uses React Query for state management and Zustand for local state
  - Runs on port 3001 (dev)

### Packages

- **`packages/client-sdk/`** - Client SDK for accessing laf from frontend applications
- **`packages/cloud-sdk/`** (`@lafjs/cloud`) - SDK provided to cloud functions at runtime
- **`packages/database-proxy/`** - Database access layer with ACL support (MongoDB/MySQL)
- **`packages/database-ql/`** - Query language abstraction
- **`packages/node-modules-utils/`** - Utilities for managing npm dependencies
- **`packages/eslint-config-laf/`** - Shared ESLint configuration

### Other

- **`cli/`** - Command-line tool for laf
- **`e2e/`** - End-to-end tests using Jest
- **`docs/`** - VitePress documentation
- **`deploy/`** - Kubernetes deployment manifests
- **`services/runtime-exporter/`** - Prometheus exporter for runtime metrics

## Development Commands

### Root Level (Monorepo)

```bash
# Install all dependencies across workspaces
npm install
# Or with lerna
lerna exec npm install --parallel

# Build all packages
npm run build
# Or
lerna run build --parallel

# Lint all packages
npm run lint
# Or
lerna run lint --parallel

# Run lint-staged (pre-commit)
npm run lint-staged
```

### Server Development

```bash
cd server/

# Install dependencies
npm install

# Development with watch mode
npm run dev
# Or
npm run watch

# Build
npm run build

# Start production
npm run start:prod

# Run tests
npm test
npm run test:watch
npm run test:cov
npm run test:e2e

# Lint
npm run lint
```

**Local development with Telepresence** (connects to Kubernetes cluster):

```bash
cd server/

# Install traffic manager
telepresence helm install

# Connect to cluster
telepresence connect -n laf-system

# Intercept traffic (creates .env file)
telepresence intercept laf-server -p 3000:3000 -e $(pwd)/.env

# Start dev server
npm run dev

# Clean up
telepresence leave laf-server
```

### Web Development

```bash
cd web/

# Install dependencies (uses pnpm or npm)
npm install
# Or
pnpm install

# Development server
npm run dev

# Build for production
npm run build

# Type check
npm run tsc

# Preview production build
npm run preview

# Lint
npm run lint
```

### Runtime Development

```bash
cd runtimes/nodejs/

# Install and build
npm install
npm run build

# Development
npm run dev
npm run watch

# Start
npm start

# Lint
npm run lint
```

**Local development with Telepresence**:

```bash
cd runtimes/nodejs/

# Connect to cluster
telepresence connect -n laf-system

# Set your test app ID
export appid=your-app-id

# Intercept app traffic
telepresence intercept $appid -p 8000:8000 -e $(pwd)/.env

# Start runtime
npm run build
npm start

# Clean up
telepresence leave $appid
```

### Package Development

```bash
# Example: database-proxy
cd packages/database-proxy/

npm install
npm run build
npm run watch  # Watch mode
npm test       # Run tests
npm run lint
```

### E2E Tests

```bash
cd e2e/

# Run all tests
npm test

# Or use the shell script
./e2e.sh
```

### CLI Development

```bash
cd cli/

npm install
npm run build
npm run watch

# Use locally
node dist/main.js
```

## Architecture Overview

### Request Flow

1. User accesses web console (`web/`) to manage applications and write functions
2. Web console communicates with API server (`server/`) via REST APIs
3. API server manages Kubernetes resources to create application instances
4. Each application gets its own runtime pod (`runtimes/nodejs/`)
5. Runtime pods execute user-defined cloud functions
6. Cloud functions use `cloud-sdk` to access database (via `database-proxy`) and storage

### Key Architecture Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [labring/laf](https://github.com/labring/laf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
