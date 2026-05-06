---
trigger: always_on
description: This repository is an **OpenAPI TypeScript Client Generator** that converts
---

# Copilot Instructions for TypeScript

## Repository Summary

This repository is an **OpenAPI TypeScript Client Generator** that converts
OpenAPI specifications into fully-typed Zod v4 schemas and type-safe REST API
clients. It generates operation-based TypeScript clients with runtime validation
capabilities, supporting OpenAPI 2.0, 3.0.x, and 3.1.x specifications.

**Repository Size**: Medium (~33 TypeScript files, ~4,500 lines of code)  
**Project Type**: CLI tool and library for code generation  
**Languages**: TypeScript (ES2022), Node.js  
**Frameworks**: Zod v4 for schema validation, Vitest for testing  
**Target Runtime**: Node.js 22+

## Build and Validation Instructions

### Prerequisites

- **Node.js**: Version 22+ (specified in `.node-version`)
- **Package Manager**: pnpm 10.14.0+ (ALWAYS use pnpm, not npm)

### Setup Commands

```bash
# Install pnpm if not available
npm install -g pnpm@10.14.0

# Install dependencies (ALWAYS run this first)
pnpm install
```

### Build Process

```bash
# Build the project (compiles TypeScript to dist/)
pnpm run build
# Build time: ~2 seconds
```

### pnpm Tasks

- `pnpm run build`: Build the project using tsup (compiles TypeScript to dist/
  without type checking)
- `pnpm run build:docs`: Generate documentation by embedding code in README.md
- `pnpm run lint`: Run oxlint with autofix on `src/`
- `pnpm run lint:check`: Run oxlint on `src/` (no autofix)
- `pnpm run format`: Format all files using oxfmt (writes changes)
- `pnpm run format:check`: Check formatting using oxfmt (no changes written)
- `pnpm run typecheck`: Run TypeScript type checking only (`tsc --noEmit`)
- `pnpm run test`: Run all tests with Vitest
- `pnpm run test:coverage`: Run tests with coverage report
- `pnpm run start`: Run the CLI from `dist/index.js`
- `pnpm run generate`: Generate client and server from test fixtures
- `pnpm run prepublishOnly`: Build docs and project before publishing
- `pnpm run release:patch`: Bump patch version and push
- `pnpm run release:minor`: Bump minor version and push
- `pnpm run release:major`: Bump major version and push

#### Additional VS Code Tasks

The workspace provides the following VS Code tasks for common workflows:

- **TypeScript Build**: `pnpm run build`
- **Run Tests**: `pnpm test`
- **Test OpenAPI 3.1 Generation**: `pnpm generate`

> **Note:** Always run `pnpm install` before any other command. Use
> `pnpm run typecheck` for type validation, and run `pnpm run lint` and
> `pnpm run format` before committing code.

**Preconditions**:

- Dependencies must be installed with `pnpm install`
- No linting is required before build

**Postconditions**:

- Creates `dist/` directory with compiled JavaScript
- All TypeScript files compiled to ES2022 modules

### Testing

```bash
# Run all tests
pnpm test
```

**Test Configuration**: Uses Vitest with Node.js environment, tests located in
`tests/`

### CLI Usage

```bash
# Generate schemas only
pnpm start generate -i <openapi-spec> -o <output-dir>

# Generate schemas and client
pnpm start generate -i <openapi-spec> -o <output-dir> --client

# Example with provided test files
pnpm start generate -i test.yaml -o ./generated --client
```

**Note**: Use single command format (`pnpm start generate`) not double-dash
format (`pnpm start -- generate`)

### Programmatic Usage

```typescript
import { generate } from "./src/core-generator/index.js";

await generate({
  input: "./openapi.yaml",
  output: "./generated",
  generateClient: true,
});
```

### Validation Steps

1. **Always run `pnpm install` before any other command**
2. **Build validation**: `pnpm run build` should complete without errors
3. **Test validation**: `pnpm test` should pass all tests
4. **CLI validation**: Test generation with `test.yaml` or `definitions.yaml`
5. **File Formatting**: `pnpm format:check`must pass
6. **Linting**: `pnpm lint:check`must pass

### Known Issues and Workarounds

- **CLI parsing**: Use `pnpm start generate` (not `pnpm start -- generate`)
- **Error handling**: Generator is robust and continues processing even with
  invalid input files

## Monorepo Layout and Architecture

### Top-Level Directory Structure

```
├── apps/                         # Main packages and applications
│   ├── craft/                    # Core OpenAPI generator package
│   │   ├── src/                  # Source code (client-generator, core-generator, etc.)
│   │   ├── tests/                # Unit and integration tests
│   │   ├── package.json          # Package manifest for craft
│   │   └── ...                   # Build, config, and docs
│   └── examples/                 # Example OpenAPI specs, generated output, and usage
│       ├── generated/            # Example generated client/server/schemas
│       ├── client-examples/      # Example client usage
│       ├── server-examples/      # Example server usage
│       └── ...                   # Benchmarks, fixtures, etc.
├── website/                      # Docusaurus documentation site
├── local/                        # Local experiments, generated files, and scratch
├── .github/                      # GitHub workflows and Copilot instructions
├── package.json                  # Monorepo root manifest (scripts, workspaces)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gunzip/apical-ts](https://github.com/gunzip/apical-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
