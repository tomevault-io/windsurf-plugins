---
trigger: always_on
description: When generating code for this repository:
---

# GitHub Copilot Instructions for zigbee-herdsman

## Priority Guidelines

When generating code for this repository:

1. **Version Compatibility**: Always respect the exact versions of Node.js, TypeScript, and libraries used in this project
2. **Codebase Patterns**: Scan the codebase for established patterns before generating code
3. **Architectural Consistency**: Maintain the layered architecture and established module boundaries
4. **Code Quality**: Prioritize maintainability, type safety, and consistency with existing patterns
5. **Testing**: Follow the established Vitest testing patterns

## Technology Stack & Versions

### Core Technologies

- **Runtime**: Node.js (CommonJS module system)
- **Language**: TypeScript 5.9.2
  - Target: ES2022
  - Module: NodeNext
  - Strict mode enabled (`strict: true`, `noImplicitAny: true`, `noImplicitThis: true`)
- **Package Manager**: pnpm 10.12.1

### Key Dependencies

- **Serial Communication**: `@serialport/stream` (^13.0.0), `@serialport/bindings-cpp` (^13.0.1)
- **Network Discovery**: `bonjour-service` (^1.3.0)
- **Utilities**: `fast-deep-equal` (^3.1.3), `debounce` (^2.2.0), `mixin-deep` (^2.0.1)
- **Zigbee Adapters**: Support for Z-Stack, EZSP/EmberZNet, deCONZ, Zigate, ZBOSS, ZOH

### Development Tools

- **Testing**: Vitest 3.2.4 with @vitest/coverage-v8
- **Code Quality**: Biome 2.2.5 (formatting, linting)
- **Build**: TypeScript compiler with incremental compilation

## Project Architecture

### Layered Structure

```
src/
├── index.ts                 # Public API exports
├── adapter/                 # Hardware adapter layer (Z-Stack, EZSP, etc.)
├── buffalo/                 # Binary data serialization/deserialization
├── controller/              # Core business logic
│   ├── controller.ts       # Main controller orchestration
│   ├── database.ts         # Persistence layer
│   ├── helpers/            # Shared utilities
│   └── model/              # Domain models (Device, Endpoint, Group)
├── models/                  # Backup and configuration models
├── utils/                   # Cross-cutting utilities
└── zspec/                   # Zigbee specification implementation
    ├── zcl/                # Zigbee Cluster Library
    └── zdo/                # Zigbee Device Objects
```

### Key Architectural Principles

1. **Separation of Concerns**: Adapter layer handles hardware communication, controller handles business logic
2. **Entity Pattern**: Device, Endpoint, Group, Entity form a hierarchy
3. **Event-Driven**: Controller extends EventEmitter for loose coupling
4. **Static Caching**: Device and Group use static Maps for singleton-like behavior
5. **Database Abstraction**: All persistence goes through `database.ts`

## Code Style & Formatting

### Biome Configuration

The project uses Biome for code quality enforcement. Key settings:

- **Indentation**: 4 spaces
- **Line Width**: 150 characters
- **Bracket Spacing**: false (e.g., `{foo}` not `{ foo }`)
- **Naming Conventions**: Flexible (camelCase, PascalCase, snake_case, CONSTANT_CASE allowed based on context)
- **No Non-Null Assertions**: Discouraged except in test files (use `!` sparingly)
- **Unused Imports**: Error level

### Import Organization

Follow this pattern observed in the codebase:

```typescript
// 1. Node.js built-ins
import assert from "node:assert";
import events from "node:events";

// 2. External dependencies
import mixinDeep from "mixin-deep";

// 3. Internal imports (relative paths, grouped by layer)
import {Adapter, type Events as AdapterEvents} from "../adapter";
import {logger} from "../utils/logger";
import * as Zcl from "../zspec/zcl";
import * as Zdo from "../zspec/zdo";

// 4. Relative imports from same layer
import Database from "./database";
import {Device, Entity} from "./model";
```

## Naming Conventions

### Variables and Properties

- **Private fields**: Use `#` prefix for true private fields: `#customClusters`, `#genBasic`
- **Internal properties**: Use `_` prefix: `_members`, `_endpoints`, `_ieeeAddr`
- **Public properties**: camelCase: `groupID`, `networkAddress`, `interviewState`
- **Constants**: SCREAMING_SNAKE_CASE in dedicated files or modules
- **Static members**: camelCase for methods, PascalCase for types

### Functions and Methods

- **Public methods**: camelCase: `addMember()`, `removeFromNetwork()`, `getDevice()`
- **Private methods**: camelCase: `toDatabaseRecord()`, `fromDatabaseEntry()`
- **Static factory methods**: camelCase: `byGroupID()`, `byIeeeAddr()`
- **Iterators**: Suffix with `Iterator`: `allIterator()`, `getDevicesIterator()`

### Classes and Types

- **Classes**: PascalCase: `Controller`, `Device`, `Endpoint`, `Group`
- **Interfaces**: PascalCase: `Options`, `DatabaseEntry`, `ConfigureReportingItem`
- **Type Aliases**: PascalCase for complex types, camelCase for simple unions
- **Enums**: PascalCase with CONSTANT_CASE or PascalCase members

### Files

- **Source files**: camelCase: `controller.ts`, `zclFrameConverter.ts`
- **Test files**: camelCase with `.test.ts` suffix: `controller.test.ts`
- **Type definition files**: camelCase: `tstype.ts`, `tstypes.ts`

## TypeScript Patterns

### Type Safety

Always use explicit types for public APIs:

```typescript
// ✅ Good: Explicit return type
public async write<Cl extends number | string>(
    clusterKey: Cl,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Koenkk/zigbee-herdsman](https://github.com/Koenkk/zigbee-herdsman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
