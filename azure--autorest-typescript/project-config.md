---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# GitHub Copilot Instructions for autorest.typescript

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

This repository focuses on the TypeSpec TypeScript emitter, which generates TypeScript client libraries from [TypeSpec](https://typespec.io/) specifications.

## Working Effectively

- **Bootstrap, build, and test the repository:**
  - `npm install -g pnpm` -- Install pnpm globally (takes ~2 minutes)
  - `PUPPETEER_SKIP_DOWNLOAD=true pnpm install` -- Install dependencies, takes ~8 seconds. NEVER CANCEL. Set timeout to 15+ minutes.
  - `pnpm build` -- Build all packages, takes ~12 seconds. NEVER CANCEL. Set timeout to 20+ minutes.
  - `pnpm format` -- Format all code, takes ~6 seconds. NEVER CANCEL. Set timeout to 10+ minutes.

- **Run tests:**
  - Unit tests: `npm run unit-test` in `packages/typespec-ts/` -- takes ~2.5 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
  - Autorest unit tests: `npm run unit-test` in `packages/autorest.typescript/` -- takes ~5 seconds. NEVER CANCEL. Set timeout to 5+ minutes.
  - RLC common unit tests: `npm run test` in `packages/rlc-common/` -- takes ~4 seconds. NEVER CANCEL. Set timeout to 5+ minutes.
  - Smoke test: `npm run smoke-test` in `packages/typespec-test/` -- takes ~10 minutes. NEVER CANCEL. Set timeout to 20+ minutes. **Wait until you see "All specs succeeded!" message.**

- **Run integration tests (from `packages/typespec-ts/`):**
  - ALWAYS run `npm run copy:typespec` first before integration tests (takes <1 second)
  - RLC Integration: `npm run integration-test-ci:rlc` -- takes ~30+ minutes. NEVER CANCEL. Set timeout to 60+ minutes.
  - Azure RLC Integration: `npm run integration-test-ci:azure-rlc` -- takes ~30+ minutes. NEVER CANCEL. Set timeout to 60+ minutes.
  - Modular Integration: `npm run integration-test-ci:modular` -- takes ~30+ minutes. NEVER CANCEL. Set timeout to 60+ minutes.
  - Azure Modular Integration: `npm run integration-test-ci:azure-modular` -- takes ~30+ minutes. NEVER CANCEL. Set timeout to 60+ minutes.
  - All integration tests: `npm run integration-test-ci` -- takes ~1+ hour. NEVER CANCEL. Set timeout to 90+ minutes.

- **Validation steps before committing:**
  - `pnpm install` -- Ensure dependencies are correctly installed (takes ~8 seconds)
  - `pnpm build` -- Build all packages and verify no build issues (takes ~12 seconds)
  - `pnpm format` -- Format code
  - `npm run lint` in `packages/typespec-ts/` -- Lint TypeSpec emitter (takes ~6 seconds)
  - `npm run check-format` in `packages/typespec-ts/` -- Check code formatting (takes ~5 seconds)

## Key Concepts

We have two main SDK styles in this repository. One is the [REST Level Client (RLC) SDK](https://devblogs.microsoft.com/azure-sdk/azure-rest-libraries-for-javascript/), and the other is the [Modular SDK](https://devblogs.microsoft.com/azure-sdk/azure-sdk-modularized-libraries-for-javascript/).

1. **REST Level Client (RLC) SDK**

   - Low-level, direct mapping to REST API operations
   - One-to-one correspondence with API endpoints
   - Follows REST Level Client patterns
   - Minimal abstraction over HTTP operations

2. **Modular SDK**

   - Higher-level abstraction over REST API operations
   - Modularized design for better organization
   - Provides a more user-friendly API surface
   - Strong type safety and IntelliSense support

## Project Structure

- `packages/typespec-ts/` - TypeSpec TypeScript emitter

  - Generates TypeScript http clients from TypeSpec definitions and this is a plugin to the TypeSpec compiler which is called by the compiler with the compiled program as output:
    - Both RLC and Modular SDK generation
    - RLC provides low-level REST client operations
    - Modular SDK offers modern, enhanced developer experience

- `packages/rlc-common/` - Common utilities for REST Level Client generation

  - Some shared utilities among RLC and Modular SDK
  - Files generation changes and utilities specific to RLC go here

- `packages/typespec-test/` - Smoke test for TypeSpec TypeScript emitter

  - Under `packages/typespec-test/test/` directory, it contains various test scenarios for validating the TypeSpec TypeScript emitter
  - For each test scenario, the `spec` folder contains the TypeSpec definitions and the `generated` folder contains the generated TypeScript client libraries
  - Each test scenario is designed to ensure the generated client libraries are correct, buildable, and function as expected
  - The test scenarios cover both RLC and Modular SDK generation

- `packages/autorest.typescript/` - AutoRest TypeScript generator
  > **Note:** @autorest/typescript is in maintenance mode and should not be used as reference nor edited unless explicitly requested.

## Validation

- ALWAYS manually validate any new code by running the complete test suite including unit tests, integration tests, and smoke tests.
- Test generated client functionality by examining the generated code in `packages/typespec-test/test/*/generated/`
- Always run `npm run format` and linting commands before committing.

## Common Network Limitations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/autorest.typescript](https://github.com/Azure/autorest.typescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
