---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EffectiveAgent is a TypeScript framework for building robust, scalable AI agents using Effect-TS. The framework provides a modular architecture with strong type safety, composable async operations, and comprehensive dependency management.

## Build Commands

**Install dependencies:**
```bash
bun install
```

**Build the project:**
```bash
bun run build
```

**Type checking:**
```bash
bun run typecheck
```

**Clean build artifacts:**
```bash
bun run clean
```

**Run tests:**
```bash
bun test
```

**Run tests with coverage:**
```bash
bun test --coverage
```

**Run single test file:**
```bash
bun test path/to/test-file.test.ts
```

**Lint code:**
```bash
bunx biome lint .
```

## Architecture Overview

### Core Service Pattern

**All services use the Effect.Service class pattern (v3.16+).** NEVER use `Context.Tag` directly or the class-based tag pattern.

```typescript
// Correct service definition
export class MyService extends Effect.Service<MyServiceInterface>()("MyService", {
  effect: Effect.gen(function* () {
    // Implementation
  })
})
```

### Service Self-Configuration

Each domain service loads its own configuration via `ConfigurationService`:
- **ModelService** - Self-configures from `models.json` via `MODELS_CONFIG_PATH`
- **ProviderService** - Self-configures from `providers.json` via `PROVIDERS_CONFIG_PATH`
- **PolicyService** - Self-configures from `policies.json` via `POLICY_CONFIG_PATH`

Services use the `bootstrap()` function to access master configuration paths from `MASTER_CONFIG_PATH`.

### AgentRuntimeService

The central orchestration layer that provides:
- **Agent lifecycle management** - Create, terminate, monitor agent actors
- **Service access** - Unified interface to ModelService, ProviderService, PolicyService
- **Message handling** - Prioritized mailbox processing with activity streaming
- **State management** - Type-safe agent state with concurrent updates via Effect.Ref

### Monorepo Structure

The project uses a Bun workspace monorepo structure:

```
EffectiveAgent/
├── src/                            # Main application code
├── packages/                       # Internal packages
│   └── effect-aisdk/              # @effective-agent/ai-sdk
└── package.json                    # Workspace configuration
```

**Internal Packages:**
- **`@effective-agent/ai-sdk`** - Standalone Effect-TS communication layer for AI operations
  - Type-safe wrappers around Vercel AI SDK
  - Message transformation utilities
  - Schema conversion utilities
  - Provider factory for creating AI provider instances
  - Error handling with Effect integration

### Path Aliases

The project uses TypeScript path aliases defined in tsconfig.json:
- `@/*` - src/*
- `@core/*` - src/services/core/*
- `@ai/*` - src/services/ai/*
- `@capabilities/*` - src/services/capabilities/*
- `@pipeline/*` - src/services/pipeline/*
- `@ea-agent-runtime/*` - src/ea-agent-runtime/*
- `@effective-agent/ai-sdk` - packages/effect-aisdk/src/index.ts

## Service Architecture

```
services/
├── ai/
│   ├── model/          # AI model definitions and capabilities
│   ├── policy/         # Usage policies and rate limiting
│   ├── provider/       # AI provider configurations and clients
│   ├── tool-registry/  # Central tool registry
│   └── tools/          # Tool execution and validation
├── core/
│   ├── configuration/  # Configuration loading and validation
│   ├── health/         # Service health monitoring
│   ├── performance/    # Performance metrics
│   └── test-utils/     # Testing utilities (effect-test-harness)
├── execution/
│   ├── orchestrator/   # Policy-enforced operation orchestration
│   └── resilience/     # Circuit breakers, retries, fallback strategies
├── capabilities/
│   └── skill/          # Modular agent skills
└── producers/
    ├── chat/           # AI chat completions
    ├── embedding/      # Vector embeddings
    ├── image/          # Image generation
    ├── object/         # Structured object generation
    ├── text/           # Text generation
    └── transcription/  # Audio transcription
```

### Agent Runtime Structure

```
ea-agent-runtime/
├── api.ts                  # Public API surface
├── service.ts              # AgentRuntimeService implementation
├── initialization.ts       # Service bootstrap and composition
├── bootstrap.ts            # Master config loading
└── types.ts                # Core type definitions
```

## Configuration

Configuration files are stored in `configuration/config/`:
- `master-config.json` - Master configuration with file system settings, logging, and service config paths
- `models.json` - AI model definitions and capabilities
- `providers.json` - AI provider configurations and API keys
- `policies.json` - Usage policies and rate limits

Environment variables for configuration paths:
- `MASTER_CONFIG_PATH` - Path to master config (default: ./configuration/config/master-config.json)
- `MODELS_CONFIG_PATH` - Loaded from master config
- `PROVIDERS_CONFIG_PATH` - Loaded from master config
- `POLICY_CONFIG_PATH` - Loaded from master config

## Effect-TS Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaulJPhilp/EffectiveAgent](https://github.com/PaulJPhilp/EffectiveAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
