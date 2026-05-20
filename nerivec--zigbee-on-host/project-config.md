---
trigger: always_on
description: When generating code for this repository:
---

# GitHub Copilot Instructions

## Priority Guidelines

When generating code for this repository:

1. **Version Compatibility**: Always detect and respect the exact versions of languages, frameworks, and libraries used in this project
2. **Codebase Patterns**: Scan the codebase for established patterns before generating any code
3. **Architectural Consistency**: Maintain layered architecture with clear boundaries between drivers, protocol layers, and utilities
4. **Code Quality**: Prioritize performance, maintainability, and security in all generated code
5. **Zero Dependencies**: Never suggest external production dependencies - this project maintains zero production dependencies by design

## Technology Version Detection

This project uses the following exact technology versions:

### Core Technologies

- **Node.js** (prefer v24)
- **TypeScript**
  - Target: esnext
  - Module: NodeNext
  - Module Resolution: NodeNext
  - Strict mode enabled
- **Package Manager**: npm

Refer to [../package.json](../package.json) for versions.

### Development Dependencies

- **@biomejs/biome**: linting and formatting
- **vitest**: testing framework
- **@vitest/coverage-v8**: coverage provider
- **serialport**: dev dependency only
- **@types/node**: typing

Refer to [../package.json](../package.json) for versions.

### Critical Version Constraints

- **NodeNext modules**: Always use `.js` extensions in imports (e.g., `from "./spinel.js"` not `from "./spinel"`)
- **Strict TypeScript**: All strict type-checking options enabled
- **No external production dependencies**: Use only Node.js built-in modules for production code

## Project Architecture

### Layered Architecture

This project follows a strict layered architecture. Refer to [../docs/architecture.md](../docs/architecture.md)

### Build Configurations

- **Development build**: `npm run build` (includes src/dev/)
- **Production build**: `npm run build:prod` (excludes src/dev/)
- Always use production build for library distribution

## Code Style Standards

### Formatting (Biome Configuration)

- **Indentation**: 4 spaces (never tabs)
- **Line width**: 150 characters maximum
- **Line endings**: LF (Unix-style)
- **Quote style**: Double quotes (`"string"` not `'string'`)
- **Self-closing elements**: Required for JSX-like syntax

### Naming Conventions

Based on actual codebase patterns:

#### TypeScript Types and Interfaces

```typescript
// Types: PascalCase
export type NetworkParameters = { ... };
export type ZigbeeAPSHeader = { ... };
export type MACCapabilities = { ... };

// Interfaces: PascalCase with descriptive names
interface AdapterDriverEventMap { ... }
export interface Logger { ... }
```

#### Enums

```typescript
// Enum names: PascalCase
// Enum members: CONSTANT_CASE (enforced by Biome)
export enum InstallCodePolicy {
    NOT_SUPPORTED = 0x00,
    NOT_REQUIRED = 0x01,
    REQUIRED = 0x02,
}

// Const enums allowed for compile-time constants
export const enum ZigbeeConsts {
    COORDINATOR_ADDRESS = 0x0000,
    BCAST_DEFAULT = 0xfffc,
    HA_ENDPOINT = 0x01,
}
```

#### Functions and Variables

```typescript
// Functions: camelCase with descriptive names
function encodeSpinelFrame(): HdlcFrame { ... }
function decodeMACHeader(): MACHeader { ... }
function makeKeyedHashByType(): KeyedHash { ... }

// Variables: camelCase
const frameCounter = 0;
let securityHeader: ZigbeeSecurityHeader;
const NS = "ot-rcp-driver"; // namespace constant

// Constants: SCREAMING_SNAKE_CASE for file-level
const HDLC_TX_CHUNK_SIZE = 256;
const SPINEL_HEADER_FLG_SPINEL = 0x02;
```

#### File Names

```typescript
// Kebab-case with descriptive names
// ot-rcp-driver.ts
// ot-rcp-parser.ts
// zigbee-aps.ts
// zigbee-nwk.ts
// zigbee-nwkgp.ts
```

### Import/Export Patterns

#### Import Order and Style

```typescript
// 1. Node.js built-in modules (with node: prefix)
import EventEmitter from "node:events";
import { readFile, writeFile } from "node:fs/promises";
import { join } from "node:path";

// 2. External dependencies (none in production code)
// (dev dependencies only in src/dev/ and test/)

// 3. Internal imports (always with .js extension)
import { SpinelCommandId } from "../spinel/commands.js";
import { decodeHdlcFrame, type HdlcFrame } from "../spinel/hdlc.js";
import { logger } from "../utils/logger.js";

// 4. Relative imports
import { OTRCPParser } from "./ot-rcp-parser.js";
```

#### Export Patterns

```typescript
// Named exports preferred
export type NetworkParameters = { ... };
export enum InstallCodePolicy { ... }
export function encodeSpinelFrame(): HdlcFrame { ... }

// Const enums for compile-time constants
export const enum ZigbeeConsts { ... }

// Type-only imports/exports when appropriate
import type { HdlcFrame } from "./hdlc.js";
export type { ZigbeeAPSHeader };
```

### Type Annotations

```typescript
// Always provide explicit return types for public functions
export function encodeSpinelFrame(frame: SpinelFrame): HdlcFrame { ... }

// Use type annotations for complex variables
const header: SpinelFrameHeader = { tid, nli, flg };

// Infer simple types
const count = 0; // inferred as number
const isValid = true; // inferred as boolean

// Use as const for literal types
const COMMANDS = {
    RESET: 0x01,
    SAVE: 0x02,
} as const;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nerivec/zigbee-on-host](https://github.com/Nerivec/zigbee-on-host) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
