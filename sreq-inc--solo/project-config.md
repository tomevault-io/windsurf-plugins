---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Solo is a cross-platform API client built with Tauri 2, React, and TypeScript. It supports HTTP, GraphQL, and gRPC requests with a desktop application interface. The application uses localStorage for persisting collections and requests.

## Development Commands

### Setup and Development
```bash
bun install                    # Install dependencies
bun tauri dev                  # Start in development mode
bun tauri build                # Build production application
```

### Testing
```bash
# Frontend tests (using Vitest via Bun)
bun run test                   # Run all frontend tests
bun run test:watch             # Run tests in watch mode
bun run test:ui                # Open Vitest UI
bun run test:coverage          # Run tests with coverage report

# Alternative: Use the test script directly
./test.sh                      # Same as bun run test

# Backend tests (Rust)
npm run test:backend           # Run Rust backend tests
```

**Note**: We use Vitest (not Bun's native test runner) because it has better support for React Testing Library and jsdom. The `bun run test` command executes Vitest via `bun x vitest`.

### Release & Publishing
```bash
# Full release build (recommended)
bun run release                # Install deps → Test → Build frontend → Build app

# Quick build (skip tests)
bun run release:quick          # Build frontend → Build app

# Manual steps
bun run build                  # Build frontend only
bun tauri build                # Build Tauri app only
```

**Build artifacts location:**
- macOS: `src-tauri/target/release/bundle/dmg/`
- Linux: `src-tauri/target/release/bundle/appimage/`
- Windows: `src-tauri/target/release/bundle/msi/`

See [BUILD.md](./BUILD.md) for detailed build instructions.

### Version Management
```bash
./version.sh                   # Update version across all config files
                              # Updates package.json, Cargo.toml, and tauri.conf.json
                              # Creates a git commit automatically
```

### Cleanup
```bash
bun run clean                  # Remove node_modules + build artifacts
bun run clean:build            # Remove build artifacts only
```

## Architecture

### Frontend (React + TypeScript)

The frontend is organized around React Context providers for state management:

- **FileContext** (`src/context/FileContext.tsx`): Manages collections (folders) and requests stored in localStorage. Each folder contains an array of requests with their configuration. Handles CRUD operations for folders and requests, auto-saves changes. The key in localStorage is the folder name, and the value is an array of `StoredFile` objects.

- **RequestContext** (`src/context/RequestContext.tsx`): Manages the current request state including method, URL, payload, authentication, and response handling. Supports HTTP, GraphQL, and gRPC request types. Invokes Tauri commands to communicate with the Rust backend for executing requests.

- **VariablesContext** (`src/context/VariablesContext.tsx`): Manages folder-scoped variables using `{{variableName}}` syntax for URL and payload substitution. Variables are stored per-folder with the key pattern `solo-variables-{folderName}` in localStorage. Also uses sessionStorage to track the current folder (`current-request-folder`).

- **ThemeContext** (`src/context/ThemeContext.tsx`): Manages light/dark theme switching with persistence in localStorage using the key `theme`.

### Backend (Rust + Tauri)

The backend is modular with separate modules for different concerns:

- **`src-tauri/src/http/mod.rs`**: Tauri commands for HTTP requests (`plain_request`, `basic_auth_request`, `bearer_auth_request`)

- **`src-tauri/src/graphql/mod.rs`**: GraphQL client implementation with introspection support. Provides commands like `graphql_request`, `graphql_basic_auth_request`, `graphql_bearer_auth_request`, and `graphql_introspection`.

- **`src-tauri/src/grpc/`**: gRPC client implementation (check this module for gRPC-related commands)

- **`src-tauri/src/auth/mod.rs`**: Authentication abstractions for Basic and Bearer auth

- **`src-tauri/src/client/mod.rs`**: HTTP client wrapper around reqwest

- **`src-tauri/src/error.rs`**: Error handling types

### Storage Architecture

Solo uses browser localStorage with specific key patterns:

- **Folders/Collections**: Stored with folder name as key, value is array of `StoredFile` objects
- **Variables**: Stored with key `solo-variables-{folderName}`, value is array of `Variable` objects
- **Request data**: Each request includes method, URL, payload, auth config, query params, GraphQL/gRPC specific fields

When working with storage, always check for the `solo-variables-` prefix to distinguish variable storage from collection storage.

### Variable Substitution

Variables use `{{variableName}}` syntax and are replaced in:
- URLs
- Request bodies
- Query parameters

The substitution happens in the frontend via `VariablesContext.replaceVariablesInUrl()` before sending requests to the backend. Variables are scoped per folder/collection.

## Key Patterns

### Adding New Request Types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sreq-inc/solo](https://github.com/sreq-inc/solo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
