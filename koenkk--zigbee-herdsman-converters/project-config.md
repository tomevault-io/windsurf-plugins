---
trigger: always_on
description: When generating code for this repository:
---

# GitHub Copilot Instructions

## Priority Guidelines

When generating code for this repository:

1. **Version Compatibility**: Always detect and respect the exact versions of languages, frameworks, and libraries used in this project
2. **Context Files**: Prioritize patterns and standards defined in the `.github/copilot` directory
3. **Codebase Patterns**: When context files don't provide specific guidance, scan the codebase for established patterns
4. **Architectural Consistency**: Maintain the modular architecture with clear separation between converters, devices, and library code
5. **Code Quality**: Prioritize maintainability, testability, and consistency in all generated code

## Technology Stack

### Core Technologies

**Language & Runtime:**
- TypeScript 5.9.3 (target: esnext, module: commonjs)
- Node.js 24.x (as specified in CI)
- ECMAScript: esnext features

**Key Language Features to Use:**
- Strict mode enabled (`alwaysStrict: true`)
- `noImplicitAny: true` and `noImplicitThis: true`
- Async/await for asynchronous operations
- Type inference and explicit typing where appropriate
- No unused imports (enforced by Biome)

**Package Manager:**
- pnpm 10.12.1 (specified in `packageManager` field)

**Key Dependencies:**
- `zigbee-herdsman` ^6.2.0 (primary Zigbee library)
- `buffer-crc32` ^1.0.0
- `iconv-lite` ^0.7.0
- `semver` ^7.7.1

**Development Tools:**
- Biome 2.2.5 (linter and formatter, replaces ESLint/Prettier)
- Vitest 3.1.1 (testing framework)
- TypeScript compiler with incremental builds

### TypeScript Configuration

Always respect these TypeScript settings:
- Module system: CommonJS (`module: "commonjs"`)
- Target: ESNext (`target: "esnext"`)
- Synthetic default imports enabled
- ES module interop enabled
- Declaration files and maps generated
- Source maps enabled
- Incremental and composite builds enabled

## Project Architecture

### Directory Structure

```
src/
  index.ts                    # Main entry point, device lookup
  indexer.ts                  # Build-time model index generator
  converters/
    fromZigbee.ts             # Zigbee → MQTT converters
    toZigbee.ts               # MQTT → Zigbee converters
  devices/
    *.ts                      # Device definitions by vendor
  lib/
    modernExtend.ts           # Modern extend system (primary API)
    exposes.ts                # Expose definitions
    types.ts                  # Type definitions
    utils.ts                  # Utility functions
    constants.ts              # Constants
    reporting.ts              # Reporting configurations
    store.ts                  # Global state management
    logger.ts                 # Logging utilities
    ota.ts                    # OTA update utilities
    light.ts                  # Light-specific utilities
    color.ts                  # Color conversion utilities
    [vendor].ts               # Vendor-specific utilities (ikea, philips, tuya, etc.)
test/
  *.test.ts                   # Test files
  utils.ts                    # Test utilities
  vitest.config.mts           # Vitest configuration
```

### Architecture Principles

1. **Device Definitions**: Each device file exports `definitions: DefinitionWithExtend[]`
2. **Converters**: Separate from/to Zigbee converters with strict interfaces
3. **Modern Extends**: Primary API for composing device functionality
4. **Vendor Libraries**: Vendor-specific logic isolated in `lib/[vendor].ts`
5. **Type Safety**: Everything is strongly typed using types from `lib/types.ts`

## Code Quality Standards

### Maintainability

**Naming Conventions:**
- **Constants**: `UPPER_SNAKE_CASE` or `camelCase` or `PascalCase` (flexible, per Biome config)
- **Variables**: `camelCase` for most variables
- **Exported definitions**: `definitions` for device arrays, `fz` for fromZigbee, `tz` for toZigbee, `e` for exposes presets, `ea` for exposes access, `m` for modernExtend
- **Namespace constants**: Always define `const NS = "zhc:modulename"` for logging context
- **Type definitions**: `PascalCase` for interfaces and types
- **Functions**: `camelCase` for functions

**File Organization:**
- Device files: Import converters, exposes, modern extends, vendor libs at top
- Use consistent import aliases: `* as fz`, `* as tz`, `* as exposes`, `* as m`, vendor-specific like `* as tuya`
- Group imports: external dependencies, then internal modules
- Export patterns: `export const definitions: DefinitionWithExtend[] = [...]`

**Code Structure:**
- Keep functions focused on single responsibilities
- Use early returns to reduce nesting
- Prefer composition over inheritance
- Use modern extends for device definitions rather than manual converter composition

### Testing

**Testing Framework:**
- Vitest 3.1.1 with V8 coverage
- Test files use `.test.ts` extension
- Located in `test/` directory

**Testing Patterns:**
- Import from vitest: `import {describe, expect, it, vi} from "vitest"`
- Use `describe` blocks for grouping related tests
- Use `expect` for assertions (not `assert` from node)
- Mock devices using `mockDevice` from `test/utils.ts`
- Test files should mirror source structure

**Test Structure:**
```typescript
import {describe, expect, it} from "vitest";

describe("Feature Name", () => {
    it("should describe expected behavior", () => {
        // Arrange

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Koenkk/zigbee-herdsman-converters](https://github.com/Koenkk/zigbee-herdsman-converters) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
