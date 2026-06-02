---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Microsoft 365 Agents SDK for JavaScript/TypeScript** (formerly known as BotFramework SDK v5). It provides a comprehensive framework for building enterprise-grade conversational agents that work across M365, Teams, Copilot Studio, Webchat, and other platforms.

The repository is a **monorepo** using npm workspaces with multiple interconnected packages.

## Repository Structure

### Core Packages (in `packages/`)

- **agents-activity**: Activity protocol types, validators, and schema definitions using Zod. Replaces `botframework-schema`.
- **agents-hosting**: Core hosting components including ActivityHandler, TurnContext, CloudAdapter, authentication, and state management. Replaces `botbuilder`.
- **agents-hosting-express**: Express.js integration for hosting agents.
- **agents-hosting-dialogs**: Dialog system for building conversational flows. Replaces `botbuilder-dialogs`.
- **agents-hosting-extensions-teams**: Teams-specific features (TaskModules, Messaging Extensions).
- **agents-hosting-storage-blob**: Azure Blob Storage adapter. Replaces `botbuilder-azure`.
- **agents-hosting-storage-cosmos**: CosmosDB storage adapter. Replaces `botbuilder-azure`.
- **agents-copilotstudio-client**: Direct-to-Engine client for interacting with Copilot Studio agents.

### Test Agents (in `test-agents/`)

Sample agents used for internal testing ("samples"). For learning examples, refer to https://github.com/Microsoft/Agents.

## Common Development Commands

### Building

```bash
# Build all packages using TypeScript project references
npm run build

# Clean dist directories and rebuild
npm run build:clean

# Build sample test agents
npm run build:samples

# Build browser bundles (runs automatically after build)
npm run build:browser
```

**Build Process**: Uses TypeScript project references defined in `tsconfig.build.json`. Each package builds independently but respects dependencies through references.

### Testing

```bash
# Run all tests using Node's built-in test runner
npm test

# Run a single test file
node --test --import tsx packages/agents-activity/test/exceptionHelper.test.ts

# Run tests for a specific package
node --test --import tsx 'packages/agents-activity/test/**/*.test.ts'
```

**Test Framework**: Uses Node.js built-in `node:test` module with `tsx` for TypeScript support. Test files follow the pattern `**/*.test.ts` and use `describe`/`it` syntax.

### Linting

```bash
# Run eslint on all files
npm run lint

# Lint a specific file
npx eslint packages/agents-activity/src/index.ts
```

**Linting**: Uses `neostandard` ESLint configuration with TypeScript support.

### Documentation

```bash
# Generate API documentation using TypeDoc
npm run docs
```

### Other Commands

```bash
# Check API compatibility
npm run compat

# Launch agents playground (interactive testing tool)
npm run play
```

## Key Architecture Concepts

### Package Dependencies

The package dependency hierarchy (simplified):
- `agents-activity` (base layer - no internal dependencies)
- `agents-hosting` (depends on `agents-activity`)
- `agents-hosting-express` (depends on `agents-hosting`)
- `agents-hosting-dialogs` (depends on `agents-hosting`)
- Storage and extension packages (depend on `agents-hosting`)

### Core Abstractions

**Activity Protocol**: All communication uses the Activity schema defined in `agents-activity`. Activities are validated using Zod schemas and can be created with `Activity.fromObject()`.

**TurnContext**: Represents a single turn of conversation. Provides access to the activity, state, and methods to send responses. Created by the adapter for each incoming activity.

**ActivityHandler**: Base class for handling different activity types (message, conversationUpdate, etc.). Provides event-based hooks like `onMessage`, `onConversationUpdate`.

**AgentApplication**: Modern application-style API for building agents. Provides a higher-level abstraction over ActivityHandler with built-in state management, routing, and AI integration capabilities.

**CloudAdapter**: Processes incoming HTTP requests, authenticates them, and creates TurnContext. Handles the communication with channels.

**Storage**: Abstraction for persisting state (conversation, user, private). Implementations exist for Memory, Blob, and Cosmos.

### Authentication

Authentication configuration is loaded from environment variables using `loadAuthConfigFromEnv()`. The SDK supports:
- JWT authentication with JWKS validation
- Microsoft Entra ID (formerly Azure AD) authentication
- Bot Framework authentication
- Copilot Studio authentication (for `agents-copilotstudio-client`)

Use `authorizeJWT()` middleware for Express applications to validate incoming requests.

### Module System

**Important**: This project uses ES6 modules (`"type": "module"` in package.json). All imports must use explicit file extensions in the built output, and the code targets Node.js 20+.

## TypeScript Configuration

- **Target**: ES2019 with ES2022 lib
- **Module**: node16 (ESM)
- **Strict mode**: Enabled
- **Project References**: Used for incremental builds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Agents-for-js](https://github.com/microsoft/Agents-for-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
