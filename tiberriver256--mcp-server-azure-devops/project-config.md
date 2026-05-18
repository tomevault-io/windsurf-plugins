---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Azure DevOps MCP Server - Development Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Prerequisites

You need to have `Node.js` and `npm` installed. Node 20 (LTS) or later is recommended for development.

## Building and Running

### 1. Install Dependencies
```bash
npm install  # Takes ~25 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
```

### 2. Build the Server
```bash
npm run build  # Takes ~5 seconds. Compiles TypeScript to `dist/` directory.
```

### 3. Run Tests
```bash
# Unit tests (no Azure DevOps credentials required)
npm run test:unit  # Takes ~19 seconds. NEVER CANCEL. Set timeout to 60+ seconds.

# Integration tests (requires Azure DevOps credentials)
npm run test:int  # Takes ~18 seconds. NEVER CANCEL. Set timeout to 60+ seconds.

# E2E tests (requires Azure DevOps credentials)
npm run test:e2e  # Takes ~6 seconds. NEVER CANCEL. Set timeout to 30+ seconds.

# All tests
npm test  # Takes ~45 seconds total. NEVER CANCEL. Set timeout to 90+ seconds.
```

### 4. Code Quality
```bash
npm run lint       # Takes ~3 seconds. Runs ESLint for code quality.
npm run lint:fix   # Auto-fix linting issues.
npm run format     # Takes ~3 seconds. Runs Prettier for code formatting.
```

### 5. Run the Server
```bash
# IMPORTANT: Always configure environment first using `.env` file (copy from `.env.example`)

npm run dev        # Development mode with auto-restart using ts-node-dev
npm run start      # Production mode - runs compiled version from `dist/index.js`
npm run inspector  # Debug mode with MCP Inspector tool
```

## Environment Setup

Copy `.env.example` to `.env` and configure Azure DevOps credentials:

```bash
cp .env.example .env
# Edit .env with your Azure DevOps credentials
```

**Required environment variables:**
```
AZURE_DEVOPS_ORG_URL=https://dev.azure.com/your-organization
AZURE_DEVOPS_AUTH_METHOD=pat  # Options: pat, azure-identity, azure-cli
AZURE_DEVOPS_PAT=your-personal-access-token  # For PAT auth
AZURE_DEVOPS_DEFAULT_PROJECT=your-project-name  # Optional
```

**Alternative setup methods:**
- Use `./setup_env.sh` script for interactive environment setup with Azure CLI
- See `docs/authentication.md` for comprehensive authentication guides
- Copy from `docs/examples/` directory for ready-to-use configurations (PAT, Azure Identity, Azure CLI)
- For CI/CD environments: Reference `docs/ci-setup.md` for secrets configuration

**Note:** The application will fail gracefully with clear error messages if credentials are missing.

## Submitting Changes

Before submitting a PR, ensure:

1. **Linting and formatting pass:**
   ```bash
   npm run lint:fix && npm run format
   ```

2. **Build succeeds:**
   ```bash
   npm run build
   ```

3. **Unit tests pass:**
   ```bash
   npm run test:unit
   ```

4. **Manual testing complete:**
   - Configure `.env` file (copy from `.env.example` and update values)
   - Test server startup: `npm run dev` (should start or fail gracefully with clear error messages)
   - Test MCP protocol using `npm run inspector` (requires working Azure DevOps credentials)

5. **Integration/E2E tests pass** (if you have Azure DevOps credentials):
   ```bash
   npm run test:int && npm run test:e2e
   ```

6. **Conventional commits:**
   ```bash
   npm run commit  # Use this for guided commit message creation
   ```

**Note:** Unit tests must pass even without Azure DevOps credentials - they use mocks for all external dependencies.

## Project Architecture

- **TypeScript** project with strict configuration (`tsconfig.json`)
- **Feature-based architecture:** Each Azure DevOps feature area is a separate module in `src/features/`
  - Example: `src/features/work-items/`, `src/features/projects/`, `src/features/repositories/`
- **MCP Protocol** implementation for AI assistant integration using `@modelcontextprotocol/sdk`
- **Test Strategy:** Testing Trophy approach (see `docs/testing/README.md`)
  - Unit tests: `.spec.unit.ts` (mock all external dependencies, focus on logic)
  - Integration tests: `.spec.int.ts` (test with real Azure DevOps APIs, requires credentials)
  - E2E tests: `.spec.e2e.ts` (test complete MCP server functionality)
  - Tests are co-located with feature code
- **Path aliases:** Use `@/` instead of relative imports (e.g., `import { something } from '@/shared/utils'`)

### Key Directories
- `src/index.ts` and `src/server.ts` - Main server entry points
- `src/features/[feature-name]/` - Feature modules
- `src/shared/` - Shared utilities (auth, errors, types, config)
- `src/clients/azure-devops.ts` - Azure DevOps client
- `tests/setup.ts` - Test configuration
- `docs/` - Comprehensive documentation (authentication, testing, tools)

## Adding a New Feature

Follow the Feature Module pattern used throughout the codebase:

1. **Create feature module directory:**
   ```
   src/features/[feature-name]/
   ```

2. **Add required files:**
   - `feature.ts` - Core feature logic
   - `schema.ts` - Zod schemas for input/output validation
   - `tool-definitions.ts` - MCP tool definitions
   - `index.ts` - Exports and request handlers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tiberriver256/mcp-server-azure-devops](https://github.com/Tiberriver256/mcp-server-azure-devops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
