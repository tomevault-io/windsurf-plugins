---
trigger: always_on
description: **ALWAYS follow these instructions first and only fallback to search or additional context if the information here is incomplete or found to be in error.**
---

# Copilot Instructions for hevy-mcp

**ALWAYS follow these instructions first and only fallback to search or additional context if the information here is incomplete or found to be in error.**

## Project Overview

- **hevy-mcp** is a Model Context Protocol (MCP) server for the Hevy Fitness API, enabling AI agents to manage workouts, routines, exercise templates, and folders via the Hevy API.
- The codebase is TypeScript (Node.js v20+), with a clear separation between tool implementations (`src/tools/`), generated API clients (`src/generated/`), and utility logic (`src/utils/`).
- API client code is generated from the OpenAPI spec using [Kubb](https://kubb.dev/). **Do not manually edit generated files.**
- **Type Safety:** The project uses Zod schema inference for type-safe tool parameters, eliminating manual type assertions and ensuring compile-time type safety.

## Working Effectively

### Bootstrap and Build Repository

Run these commands in order to set up a working development environment (Corepack is bundled with Node.js v20+, so run `corepack use pnpm@10.22.0` once per machine if pnpm isn't available):

1. **Install dependencies:**

   ```bash
   pnpm install
   ```

   - Takes approximately 30 seconds. NEVER CANCEL - set timeout to 60+ seconds.

2. **Build the project:**

   ```bash
   pnpm run build
   ```

   - Takes approximately 3-5 seconds. TypeScript compilation via tsdown.
   - Always build before running the server or testing changes.

3. **Run linting/formatting:**

   ```bash
   pnpm run check
   ```

   - Takes less than 1 second.
   - **EXPECTED WARNING:** Biome schema version mismatch warning is normal and can be ignored.

### Testing Commands

4. **Run unit tests only:**

   ```bash
   pnpm vitest run --exclude tests/integration/**
   ```

   - Takes approximately 1-2 seconds. NEVER CANCEL.
   - This is the primary testing command for development.

5. **Run integration tests (requires API key):**

   ```bash
   pnpm vitest run tests/integration
   ```

   - **WILL FAIL** without valid `HEVY_API_KEY` in `.env` file (by design).
   - Integration tests require real API access and cannot run in sandboxed environments.

6. **Run all tests:**

   ```bash
   pnpm test
   ```

   - Takes approximately 1-2 seconds for unit tests only (without API key).
   - **WILL FAIL** if `HEVY_API_KEY` is missing due to integration test failure (by design).

### API Client Generation

7. **Regenerate API client from OpenAPI spec:**

   ```bash
   pnpm run build:client
   ```

   - Takes approximately 4-5 seconds. NEVER CANCEL.
   - **EXPECTED WARNINGS:** OpenAPI validation warnings about missing schemas are normal.
   - Always run this after updating `openapi-spec.json`.

8. **Validate OpenAPI spec:**

   ```bash
   pnpm run validate:openapi
   ```

   - Takes less than 1 second.
   - Uses IBM OpenAPI Validator with Spectral ruleset (`.spectral.yaml`).
   - Validates `openapi-spec.json` against OpenAPI 3.0 specification.
   - **EXPECTED WARNINGS:** Since this is an external API spec from Hevy, some warnings are expected and acceptable.

### Server Operations

9. **Development server (with hot reload):**

   ```bash
   pnpm run dev
   ```

   - **REQUIRES:** Valid `HEVY_API_KEY` in `.env` file or will exit immediately.
   - Server runs indefinitely until stopped.

10. **Production server:**

```bash
pnpm start
```

- **REQUIRES:** Valid `HEVY_API_KEY` in `.env` file or will exit immediately.
- Must run `pnpm run build` first.

## Commands With Known Environment Limitations

### Known Failing Commands

- **`pnpm run export-specs`**: Fails with network error (`ENOTFOUND api.hevyapp.com`) in sandboxed environments.
- **`pnpm run inspect`**: MCP inspector tool - may timeout in environments without proper MCP client setup.

Only list commands here that are known to be flaky or unsupported in some
environments. Other documented commands (including `pnpm run check:types`) are
expected to succeed locally; treat failures as issues to fix rather than
environmental flakiness. See `README.md` for the canonical list of commands.

`pnpm run check:types` is expected to pass locally before opening a PR; see the
"Type checking validation" section below.

## Environment Setup

### Required Environment Variables

Create a `.env` file in the project root with:

```env
HEVY_API_KEY=your_hevy_api_key_here
```

**CRITICAL:** Without this API key:

- Servers will not start
- Integration tests will fail (by design)
- API client functionality cannot be tested

### Node.js Version

- **Supported:** Node.js >= 20
- **Recommended:** Use the exact version pinned in `.nvmrc` (CI uses this exact version)
- If you use `nvm`, run `nvm use` in the repo root to match `.nvmrc`
- Use `node --version` to verify current version

## Validation After Changes

### Manual Testing Scenarios

Always perform these validation steps after making changes:

1. **Build validation:**

   ```bash
   pnpm run build
   ```

   - Must complete successfully without errors.

2. **Unit test validation:**

   ```bash
   pnpm vitest run --exclude tests/integration/**
   ```

   - All unit tests must pass.

3. **Code style validation:**

   ```bash
   pnpm run check

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisdoc/hevy-mcp](https://github.com/chrisdoc/hevy-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
